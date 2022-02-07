---
published: true
---
## How to Connect Databricks VNet Injected Cluster to Synapse Dedicated Pools

This blog focusses on how [Azure VNet Injected Databricks Cluster](https://docs.microsoft.com/en-us/azure/databricks/administration-guide/cloud-configurations/azure/vnet-inject) could connect to a Synapse Dedicated pool ,which dis-allows "any-ip" connectivity and also has not permited azure services to connect to it by default.



### Pre-requisites:

- [Azure Synapse Workspace](https://docs.microsoft.com/en-us/azure/synapse-analytics/quickstart-create-workspace)
- [Azure VNet Injected Databricks Cluster](https://docs.microsoft.com/en-us/azure/databricks/administration-guide/cloud-configurations/azure/vnet-inject) 
- An IDE of your choice , for this blog i have used [VS Code](https://code.visualstudio.com/Download)
- [Azure KeyVault](https://docs.microsoft.com/en-us/azure/key-vault/general/quick-create-cli).

### Scenario : 

This is a common question which popped up at my customers, on how could we connect from databricks towards synapse dedicated pools.

To address this an architecture on how the connectivity would work is provided here :

[Connectivity Architecture](/images/databricks.png)


Steps in this project :

1. Create Synapse Workspace and restrict connectivity to Synapse Pools
2. Deploy Databricks Cluster inside your own VNet
3. Create a Private Endpoint For Synapse Dedicated Pool
4. Connect from Databricks to Synapse Dedicated Pools using Private Endpoint


### Steps Elaborated :

### 1. Create Synapse Workspace and restrict connectivity to Synapse Pools

<TBD>

### 2.Deploy Databricks Cluster inside your own VNet

<TBD>

### 3.Create a Private Endpoint For Synapse Dedicated Pool

<TBD>


### 4.Connect from Databricks to Synapse Dedicated Pools using Private Endpoint

<TBD>