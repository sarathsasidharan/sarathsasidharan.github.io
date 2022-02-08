---
published: true
---
## Connect Databricks VNet Injected Cluster to Synapse Dedicated Pools

This blog focusses on how [Azure VNet Injected Databricks Cluster](https://docs.microsoft.com/en-us/azure/databricks/administration-guide/cloud-configurations/azure/vnet-inject) could connect to a Synapse Dedicated pool ,which dis-allows "any-ip" connectivity and also has not permited azure services to connect to it by default.



### Pre-requisites:

- [Azure Synapse Workspace](https://docs.microsoft.com/en-us/azure/synapse-analytics/quickstart-create-workspace)
- [Azure VNet Injected Databricks Cluster](https://docs.microsoft.com/en-us/azure/databricks/administration-guide/cloud-configurations/azure/vnet-inject) 
- An IDE of your choice , for this blog i have used [VS Code](https://code.visualstudio.com/Download)
- [Azure KeyVault](https://docs.microsoft.com/en-us/azure/key-vault/general/quick-create-cli).

### Scenario : 

This is a common question which popped up at my customers, on how could we connect from vnet injected databricks towards a network restricted synapse dedicated pool.

To address this an architecture on how the connectivity would work is provided here :

![Connectivity Architecture](/images/databricks.png)


Steps in this project :

1. Create Synapse Workspace and restrict connectivity to Synapse Pools
2. Deploy Databricks Cluster inside your own VNet
3. Create a Private Link For Synapse Dedicated Pool
4. Connect from Databricks to Synapse Dedicated Pools using Private Link.


### Steps Elaborated :

### 1. Create Synapse Workspace and restrict connectivity to Synapse Pools

During this step , setup a new [Synapse workspace](https://docs.microsoft.com/en-us/azure/synapse-analytics/quickstart-create-workspace).

Next step is to restrict connectivity and remove our default network rules which allow every machine to access the workspace ( this is a default setting).

Under this remove the default rule which gives access to all IPs.

Disable the access to all azure services , by de-selecting the checkbox. 

![network restriction](/images/network_restriction.PNG)


### 2.Deploy Databricks Cluster inside your own VNet

Deploy a databricks cluster inside your own VNet using [Vnet Injection](https://docs.microsoft.com/en-us/azure/databricks/scenarios/quickstart-create-databricks-workspace-vnet-injection#:~:text=%20Create%20an%20Azure%20Databricks%20workspace%20%201,Networking%20%3E%20and%20apply%20the%20following...%20See%20More.)

Define a VNet where you would like to configure the subnets for databricks.

![vnet ](/images/vnet.png)

In order to do this , you will need to create 2 dedicated subnets per databricks workspace . One is a private subnet and another a public subnet. 

The private subnet is the address range used by the VMs which would be deployed during databricks cluster creation. The public IPs inside the public subnet are used for the communication between the data plane and the control plane.

![subnets](/images/subnets.PNG)

Databricks creates a managed resource group into which a storage account is created.  Databricks will be using the VNets supplied during the databricks deployment (customer VNet) . This is because we are using VNet Injection and not the default configuration. 

Databricks delegates the workpace to the subnet , which means the databricks workspace service could do changes on these subnets. There will also be intent policies which would be created to prevent users from altering the network configurations on the NSG inbound and outbound security rules.

If this is not done ,  it can sabotage the communication from the data plane of databricks to the control pane managed by Microsoft. This is essential to maintain the SLAs which are provided by the cloud provider.

While creating a new cluster , the following resources are also deployed into the managed resource group :

- Disks
- Network Interface  / NICs
- Public IP Addresses
- Virtual Machines ( Nodes in the spark cluster)

![external resources](/images/ext_resources.png)

These resources are automatically cleaned up when the cluster is terminated.

Thes VMs are attached to the private subnet which is provided during deployment. 

### 3.Create a Private Link For Synapse Dedicated Pool

In order to communicate from the databricks nodes to a dedicated SQL Pool instance , we will use [private links](https://docs.microsoft.com/en-us/azure/synapse-analytics/security/how-to-connect-to-workspace-with-private-links).

This can be achieved using 2 methods :

- Deploy Synapse Private Link in a second VNet and leverage VNet peering to help databricks clusters talk to synapse workspace dedicated pools
- Deploy Synapse Private Link in the same VNet in a different Subnet. 

** Please note that you cannot deploy a private link endpoint into a delegated vnet , so you cannot deploy the synapse dedicated private link endpoint into the subnets delegated to the databricks service.

For this blog , to keep it simple we are using the latter. We will create a new Subnet where the private link endpoint can be deployed. This subnet however belongs to the same VNet used by Databricks in the previous step.

![private link](/images/private_link.PNG)

This is the third subnet created to deploy the Synapse private link endpoint.

![synapse subnet](/images/synapse_subnet.PNG)

Since they are within the same VNet , communication will happen automatically without having to explicitly add the subnet range (of databricks) into the synapse workspace firewall rule.

### 4.Connect from Databricks to Synapse Dedicated Pools using Private Endpoint

The last part of this demo is to connect from the Databricks workspace deployed inside Customer VNet (VNet injected) to the synapse workspace deployed inside the customer VNet.

First step is to get all the secrets inside a key-vault backed [databricks scope](https://docs.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes#akv-ss). 

Store your synapse password used by the jdbc connection and account keys inside the azure keyvault.

![key vault](/images/keyvault.PNG)

Create a new notebook and test out the connectivity , this is leveraging the synapse connector to connect to the dedicated pools.

Retrieve the values from the key-vault backed databricks secret scope. Using the connector load the data into the dataframe and display the result to check if the connection was succesful.

![databricks notebook](/images/databricks_connect.PNG)

Code used in the notebook

Do make sure to replace the scope name "synapsekeys01" with the name of the scope you create in databricks.

```
dwDatabase = dbutils.secrets.get(scope="synapsekeys01", key="databricksSynapseDatabase")
dwServer = dbutils.secrets.get(scope="synapsekeys01", key="databricksSynapseServer")
dwUser = dbutils.secrets.get(scope="synapsekeys01", key="databricksSqlUser")
dwPass = dbutils.secrets.get(scope="synapsekeys01", key="databricksSynapsePoolsPwd")
dwJdbcPort = "1433"
dwJdbcExtraOptions = "encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.sql.azuresynapse.net;loginTimeout=30;"
sqlDwUrl = "jdbc:sqlserver://"+ dwServer + ".sql.azuresynapse.net:"+dwJdbcPort+";database=" + dwDatabase + ";user="+dwUser+";password="+dwPass
blobStorage = dbutils.secrets.get(scope="synapsekeys01", key="databricksBlobStorageName")
blobContainer = "databricks"
blobAccessKey = dbutils.secrets.get(scope="synapsekeys01", key="databricksBlobKey")
tempDir = "wasbs://"+blobContainer+"@"+blobStorage+"/tempDirs"
acntInfo = "fs.azure.account.key"+blobStorage
```

Make connection towards the synapse dedicated pools.

```
# Get some data from an Azure Synapse table.
synapse_table = spark.read \
  .format("com.databricks.spark.sqldw") \
  .option("url", sqlDwUrl) \
  .option("dbtable", "customer") \
  .option("forwardSparkAzureStorageCredentials", "True") \
  .option("tempdir", tempDir) \
  .load()


display(synapse_table.limit(10))

```

#### Issues Encountered :

```
Caused by: com.microsoft.sqlserver.jdbc.SQLServerException: Cannot open server '<Synapse Server Name>' requested by the login. Client with IP address '<Public IP of databricks>' is not allowed to access the server.  To enable access, use the Windows Azure Management Portal or run sp_set_firewall_rule on the master database to create a firewall rule for this IP address or address range.  It may take up to five minutes for this change to take effect. ClientConnectionId:<ID>
	at com.microsoft.sqlserver.jdbc.SQLServerException.makeFromDatabaseError(SQLServerException.java:262)
	at com.microsoft.sqlserver.jdbc.TDSTokenHandler.onEOF(tdsparser.java:283)
	at com.microsoft.sqlserver.jdbc.TDSParser.parse(tdsparser.java:129)
```

### Issue : The hostaname used inside the jdbc connection was wrong 

Resolution : Check if you are using the right hostname suffix ".sql.azuresynapse.net"
