---
published: false
---
## Automate Extraction and Deployment of  Synapse Dedicated Environments using Azure DevOps

There are many scenarios where you want to re-create your synapse dedicated pool environment into another dedicated pool.
Your company may want to copy the environment without the data to recreate an environment.

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

## Extract Dacpac From Source Dedicated Pool

![figure1](/images/extract.png)

This represents the flow of how the azure devops pipeline extracts the dacpac from the source dedicated pool.

# Code snippet for extraction of the source environment 


https://github.com/sarathsasidharan/sarathsasidharan.github.io/blob/5eb8387e685fc3365ae26ac7f2d5d956755e3770/code/sqlpackage/deploysql.yml#L11-L23

Once the pipeline within Azure DevOps is triggered. The secrets (keys / connection details etc.) of the source synapse dedicated pool are retrieved. This is stored within [variable groups](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=yaml) of Azure DevOps. Variable Groups are used to store values and secrets which need to be passed into the pipelines. Variable groups can be linked to azure key vault.

Sensitive identity and connection details locked inside an azure [key vault](https://docs.microsoft.com/en-us/azure/key-vault/general/basic-concepts) which is an HSM Solution on azure.

The secrets have to be  stored inside an azure keyvault as shown below.

![figure2](/images/kv.png)

Variables defined inside the variable group inside the pipeline can be refered using the $(<variable_name>) sytnax as seen in the code snippet above.

![figure3](/images/vg.png)

After the details have been extracted the pipeline , next goes towards the first activity in the pipeline which is a task. A task is an atomic block in a pipeline which is a pre-packaged script that performs the activity which needs to be executed. In our scenario, this is to extract the environment as an artifact. We use the pre-built task named  [SQL Package activity](https://docs.microsoft.com/en-us/sql/tools/sqlpackage/sqlpackage-pipelines?view=sql-server-ver16) to achieve this.

This task connects to the Syanpse Dedicated Pools and starts extracting the dacpac , which contains all the information needed to recreate this environment on a different pool. All these tasks are run on VMs which are called [Azure Pipeline Agents](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops&tabs=browser) /build agents. The resulting dacpac extracted from the source dedicated pool is written into the local storage of the build agent. If you are using separate pipelines to extract and deploy the artifact then you need to store this artifact in an [azure artifact](https://docs.microsoft.com/en-us/azure/devops/artifacts/start-using-azure-artifacts?view=azure-devops). This artifact can then later be retrieved in the second pipeline , to deploy the artifact. In this scenario we have just one single pipeline to extract and deploy , hence we will refer to the local drive of the build agent.

## Deploy Dacpac to target Dedicated Pool

![figure1](/images/deploy.png)

This flow represents on how the azure devops pipeline deploys the extracted dacpac from the source to the target dedicated pool.

# Code snippet for deployment to the target environment 

The SQL Package action for deployment is quite similar to the pervious task , except the DeploymentAction. By default this is deploy , so the dacpac provided is deployed to the target environment specified.

https://github.com/sarathsasidharan/sarathsasidharan.github.io/blob/1da903ef9047cec418796f555a7ee861d45bfef4/code/sqlpackage/deploysql.yml#L26-L37

The extracted artifact from the source dedicated pool (previous step) is picked and  deployed in the target dedicated pool. Second Service connection is used to connect to the second subscription aka the target subscription.The credentials for the sink dedicated pool are picked up from the key vault , via the variable groups (as discussed in the previous step). 

A connection is established to the sink dedicated  pool and the extracted dacpac is deployed. After a succesful deployment all objects in the source pool will be visible in the sink pool.

The entire code snippet used is shared here. This is a yaml file which represents the pipleine which extracts from the source and deploys to a target pool.
https://github.com/sarathsasidharan/sarathsasidharan.github.io/blob/1da903ef9047cec418796f555a7ee861d45bfef4/code/sqlpackage/deploysql.yml#L1-L37



