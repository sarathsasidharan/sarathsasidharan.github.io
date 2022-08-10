---
published: false
---
## Automate Extraction and Deployment of  Synapse Dedicated Environments using Azure DevOps

There are many scenarios where you want to re-create your synapse dedicated pool environment in another environment.Your company may want to copy the environment without the data to recreate an environment for development / other purposes.

While dealing with this on restrictive environments , performing this operation needs higher rights.

These operations have to be controlled and automated if possible to make sure :

  1. Its easy to use
  2. Control is in place
  3. Monitoring is in check
  4. Secure usage

In this blog we are going to use the [SQL Package activity](https://docs.microsoft.com/en-us/sql/tools/sqlpackage/sqlpackage-pipelines?view=sql-server-ver16) as a task in azure devops to help us achieve this.

### High Level Architecture 

![Replicate Environments](/images/sql-package.png)

This scenario imitates a dedicated hotfix subscription , which is used to get the current environment to recreate / test issues happening on 
the live environment. Azure DevOps is used to automatically extract the deployment artifact [dacpac](https://docs.microsoft.com/en-us/sql/relational-databases/data-tier-applications/data-tier-applications?view=sql-server-ver16)

In this setup we use the link between [azure devops and azure key vault](https://docs.microsoft.com/en-us/azure/devops/pipelines/release/azure-key-vault?view=azure-devops&tabs=yaml) to store secrets / connection strings which secure the pipeline.

Two separate resource groups have been defined , security resource group which contains the key-vault and the analytics-rg which contains the synapse workspace and the storage account.

The pipleine to create the package and deploy it is built within azure devops.

### Workflow Explained

The trigger for this workflows starts with an Azure DevOps Pipeline. In this scenario 2 [service connections](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml) need to be created which have rights to extract the warehouse artifacts from Synapse Dedicated pool in the source subscription and deploy the same in a sink subscription.



