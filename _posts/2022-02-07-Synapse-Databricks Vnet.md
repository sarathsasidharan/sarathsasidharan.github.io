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

This is a common question which popped up at my customers, on how could we connect from databricks towards synapse dedicated pools.

To address this an architecture on how the connectivity would work is provided here :

![Connectivity Architecture](/images/databricks.png)


Steps in this project :

1. Create Synapse Workspace and restrict connectivity to Synapse Pools
2. Deploy Databricks Cluster inside your own VNet
3. Create a Private Link For Synapse Dedicated Pool
4. Connect from Databricks to Synapse Dedicated Pools using Private Endpoint


### Steps Elaborated :

### 1. Create Synapse Workspace and restrict connectivity to Synapse Pools

During this step , setup a new [Synapse workspace](https://docs.microsoft.com/en-us/azure/synapse-analytics/quickstart-create-workspace).

Next step is to restrict connectivity and remove our default network rules which allow every machine to access the workspace ( this is a default setting).

Under this remove the default rule which gives access to all IPs.

Disable the access to all azure services , by de-selecting the checkbox. 

![network restriction](/images/network_restriction.PNG)


### 2.Deploy Databricks Cluster inside your own VNet

Deploy a databricks cluster inside your own VNet using [Vnet Injection](https://docs.microsoft.com/en-us/azure/databricks/scenarios/quickstart-create-databricks-workspace-vnet-injection#:~:text=%20Create%20an%20Azure%20Databricks%20workspace%20%201,Networking%20%3E%20and%20apply%20the%20following...%20See%20More.)

In order to do this , you will need to create 2 dedicated subnets per databricks workspace . One is a private subnet and another a public subnet. 

Databricks creates a managed resource group intow which a storage account is created.  Databricks will be using the VNets supplied during the databricks deployment (customer VNet) . This is because we are using VNet Injection and not the default configuration. 

Databricks delegates the workpace to the subnet , which means the databricks workspace service could do changes on these subnets. There will also be intent policies which would be created to prevent users from altering the network configurations on the NSG inbound and outbound security rules.

If this is not done ,  it can sabotage the communication from the data plane of databricks to the control pane managed by Microsoft. This is essential to maintain the SLAs which are provided by the cloud provider.

While creating a new cluster , the following resources are also deployed into the managed resource group :

- Disks
- Network Interface  / NICs
- Public IP Addresses
- Virtual Machines ( Nodes in the spark cluster)
![external resources](/images/ext_resources.png)

These resources are automatically cleaned up when the cluster is terminated.

Thes VMs are attached to the private subnet which is provided during deployment. The public IPs inside the public subnet are used for the control pane communication.

### 3.Create a Private Endpoint For Synapse Dedicated Pool

<TBD>


### 4.Connect from Databricks to Synapse Dedicated Pools using Private Endpoint

<TBD>