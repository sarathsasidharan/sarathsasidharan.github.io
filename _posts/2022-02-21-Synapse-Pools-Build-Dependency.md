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
