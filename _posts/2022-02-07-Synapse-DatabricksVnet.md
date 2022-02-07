---
published: true
---
## How to Connect Databricks VNet Injected Cluster to Synapse Dedicated Pools

This blog focusses on how could an [Azure VNet Injected Databricks Cluster](https://docs.microsoft.com/en-us/azure/databricks/administration-guide/cloud-configurations/azure/vnet-inject) could connect to a Synapse Dedicated pool ,which dis-allows "any-ip" connectivity and also has not permited azure services to connect to it. 



### Pre-requisites:

- [Azure Synapse Workspace](https://docs.microsoft.com/en-us/azure/synapse-analytics/quickstart-create-workspace)
- [Azure VNet Injected Databricks Cluster](https://docs.microsoft.com/en-us/azure/databricks/administration-guide/cloud-configurations/azure/vnet-inject) 
- An IDE of your choice , for this blog i have used [VS Code](https://code.visualstudio.com/Download)
- [Azure KeyVault](https://docs.microsoft.com/en-us/azure/key-vault/general/quick-create-cli).

### Scenario : 

Steps in this project :

1. Create Synapse Workspace and restrict connectivity to Synapse Pools
2. Deploy Databricks Cluster inside your own VNet
3. Create a Private Endpoint For Synapse Dedicated Pool
4. Connect from Databricks to Synapse Dedicated Pools using Private Endpoint


### Steps Elaborated :

### 1. Create Synapse Workspace and restrict connectivity to Synapse Pools

Let us take the example of a sprint developement. As a developer A we create a new branch of our Synapse Workspace to start working on a feature which needs to be developed in this sprint.

First step is to create a new branch , this can be done from synapse workspace in the sandbox environment. Point to note , on the sandbox environment there is one synapse workspace deployed , where developers have Synapse Administrator Access.

![Create Branch](/images/branching.PNG)

Once the branch has been created , the developer clones the project into his/her IDE of choice , in this case we use VS Code. Azure DevOps has a feature to directly clone this project into VSCode. Git Clone , could also be used to get this project locally into your IDE.

![Clone Project](/images/clone_project.PNG)

### 2.Deploy Databricks Cluster inside your own VNet

After the branch is checked out for developement , the next step is for the developer to start working on the feature assinged in the current sprint.

Taking an example of 3 artifacts being developed in this sprint , feature1 and feature2,feature3. Let's assume that feature1 is a functionality in spark , which is written inside a spark notebook and the second feature is a SQL Script which creates a new table and a stored procedure and feature3 consists of a mapping dataflow. In this situation we would like to check in 3 features which are all different scopes , into this single release.

For this release , developer 1 has created 3 features for Release 1.0 . They are namely:

1. Spark notebook to perform an ELT and create a spark table 
2. A SQL Script to create a new table and populate the data into the data mart
3. A Data Flow to copy the data from staging area into the data mart and the corresponding trigger

These generate 4 artifacts in our workspace, namely a spark notebook , a sql script , dataflow and a trigger file.

![Features](/images/Features.PNG)

### 3.Create a Private Endpoint For Synapse Dedicated Pool

After the feature have been created the next step is to create unit tests for these features.  We have 3 features :

1. Pyspark script for extraction of raw to clean data
2. Data Flow to get the data into a Kimball model 
3. SQL objects 

For testing , pyspark scripts we can use the pytest package and run asserts on the logic / functions.
For SQL Objects the idea would be to leverage tsql-t and [sql server unit tests](https://docs.microsoft.com/en-us/sql/ssdt/walkthrough-creating-and-running-a-sql-server-unit-test?view=sql-server-ver15)

![Tests](/images/Tests.PNG)


### 4.Connect from Databricks to Synapse Dedicated Pools using Private Endpoint

Once the tests have been defined developers would like to push these 3 features into the main branch. In order to maintain sanity on the main branch , the need to have a pull request validation pipeline is essential.

In order to to do this , a PR validation pipeline needs to be created in Azure DevOps.
