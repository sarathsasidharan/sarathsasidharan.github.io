---
published: false
---
## Automate Extraction and Deployment of  Synapse Dedicated Environments using Azure DevOps

There are many scenarios where you want to re-create your synapse dedicated pool environment in another environment.Your company may want to copy the environment without the data to recreate an environment.

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

The trigger for this workflow starts with an Azure DevOps Pipeline. In this scenario two [service connections](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml) need to be created which have rights to extract the warehouse artifacts from Synapse Dedicated pool in the source subscription and deploy the artifact in a sink subscription. A service connection is a connection object which is used by Azure DevOps to connect to azure.Under the hood , the service connections are refering to [service principals](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals) which have access to the underlying subscription for reading and writing to the target dedicated pool. 

![figure1](/images/extract.png)

# Code snippet for extraction of the source environment 

https://github.com/sarathsasidharan/sarathsasidharan.github.io/blob/b822ce588b1328abf73cb6e37269d2326223b261/code/sqlpackage/deploysql.yml#L1-L20

Once the pipeline within Azure DevOps is triggered. The secrets (keys / connection details etc.) of the source synapse dedicated pool are retrieved. This is stored within [variable groups](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=yaml) of Azure DevOps. Variable Groups are used to store values and secrets which need to be passed into the pipelines. Variable groups can be linked to azure key vault.This makes is quite secure , so we have the sensitive identity and connection details locked inside an azure [key vault](https://docs.microsoft.com/en-us/azure/key-vault/general/basic-concepts) which is an HSM Solution on azure.

After the details have been extracted the pipeline , next goes towards the first activity in the pipeline which is a task. A task is an atomic block in a pipeline which is a pre-packaged script that performs the activity which needs to be executed. In our scenario, this is to extract the environment as an artifact. We use the pre-built task named  [SQL Package activity](https://docs.microsoft.com/en-us/sql/tools/sqlpackage/sqlpackage-pipelines?view=sql-server-ver16) to achieve this.

This task connects to the Syanpse Dedicated Pools and starts extracting the dacpac , which contains all the information needed to recreate this environment on a different pool. All these tasks are run on VMs which are called [Azure Pipeline Agents](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops&tabs=browser) /build agents. The resulting dacpac extracted from the source dedicated pool is written into the local storage of the build agent. If you are using separate pipelines to extract and deploy the artifact then you need to store this artifact in an [azure artifact](https://docs.microsoft.com/en-us/azure/devops/artifacts/start-using-azure-artifacts?view=azure-devops). This artifact can then later be retrieved in the second pipeline , to deploy the artifact. In this scenario we have just one single pipeline to extract and deploy , hence we will refer to the local drive of the build agent.

![figure1](/images/deploy.png)

# Code snippet for deployment to the target environment 
https://github.com/sarathsasidharan/sarathsasidharan.github.io/blob/b822ce588b1328abf73cb6e37269d2326223b261/code/sqlpackage/deploysql.yml#L22-L33

After the extraction of the dacpac is complete , the next task in the pipeline is triggered. This has to pick up the extracted artifact from the source dedicated pool and deploy it in a sink dedicated pool. This is also achieved using a SQL Package activity. Second Service connection is used to connect to the second subscription.The credentials for the sink dedicated pool are picked up from the key vault , via the variable groups (as discussed in the previous step). 

A connection is established to the sink dedicated  pool and the extracted dacpac is deployed. After a succesful deployment all objects in the source pool will be visible in the sink pool.


