---
published: false
---
## Automate Extraction and Deployment of  Synapse Dedicated Environments using Azure DevOps

There are many scenarios where you want to re-create your synapse dedicated pool environment in another environment.

Your company may want to copy the environment without the data to recreate an environment for development / other purposes.

While dealing with this on restrictive environments , performing this operation needs higher rights.

These operations have to be controlled and automated if possible to make sure :

 (a) Its easy to use
 (b) Control is in place
 (c) Monitoring is in check
 (d) Secure usage

In this blog we are going to use the [SQL Package activity](https://docs.microsoft.com/en-us/sql/tools/sqlpackage/sqlpackage-pipelines?view=sql-server-ver16) to help us achieve this.