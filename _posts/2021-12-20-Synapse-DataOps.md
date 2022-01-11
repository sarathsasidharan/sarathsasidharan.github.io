---
published: true
---
## How to Implement Synapse CI / CD for Data Lakes

In this post we will cover an end to end synapse CI / CD implemenation for Data Lakes for enterprise deployments. 

### Pre-requisites:

- [Azure Synapse Workspace](https://docs.microsoft.com/en-us/azure/synapse-analytics/quickstart-create-workspace)
- [Azure DevOps Project](https://docs.microsoft.com/en-us/azure/devops/organizations/projects/create-project?view=azure-devops&tabs=preview-page) 
- An IDE of your choice , for this blog i have used [VS Code](https://code.visualstudio.com/Download)

### Scenario : 

Steps in this project :

1. [Link Synapse Workspace Created to a Azure DevOps Repository](https://docs.microsoft.com/en-us/azure/synapse-analytics/cicd/source-control)
2. Developers Create a new branch to work on , this is done on the sandbox environment
3. Developers check in changes and raise a Pull Request 
4. Pull Request is merged into the release branch 
5. Release Branch is Deployed into the Dev Environment using Azure DevOps Pipelines 
6. Frozen Artifacts are deployed into Acceptance and Later production after integation tests have passed

### Steps Elaborated :

#### Developers Create Branch 

Let us take the example of a sprint developement. As a developer A we create a new branch of our Synapse Workspace to start working on a feature which needs to be developed in this sprint.

First step is to create a new branch , this can be done from synapse workspace in the sandbox environment. Point to note , on the sandbox environment there is one synapse workspace deployed , where developers have Synapse Administrator Access.

![Create Branch](/images/branching.PNG)

Once the branch has been created , the developer clones the project into his/her IDE of choice , in this case we use VS Code. Azure DevOps has a feature to directly clone this project into VSCode. Git Clone , could also be used to get this project locally into your IDE.

![Clone Project](/images/clone_project.PNG)

#### Developers Create Features 

After the branch is checked out for developement , the next step is for the developer to start working on the feature assinged in the current sprint.

Taking an example of 3 artifacts being developed in this sprint , feature1 and feature2,feature3. Let's assume that feature1 is a functionality in spark , which is written inside a spark notebook and the second feature is a SQL Script which creates a new table and a stored procedure and feature3 consists of a mapping dataflow. In this situation we would like to check in 3 features which are all different scopes , into this single release.

For this release , developer 1 has created 3 features for Release 1.0 . They are namely:

1. Spark notebook to perform an ELT and create a spark table 
2. A SQL Script to create a new table and populate the data into the data mart
3. A Data Flow to copy the data from staging area into the data mart and the corresponding trigger

These generate 4 artifacts in our workspace, namely a spark notebook , a sql script , dataflow and a trigger file.

![Features](/images/Features.PNG)

#### Developers Create Tests 

After the feature have been created the next step is to create unit tests for these features.  We have 3 features :

1. Pyspark script for extraction of raw to clean data
2. Data Flow to get the data into a Kimball model 
3. SQL objects 

For testing , pyspark scripts we can use the pytest package and run asserts on the logic / functions.
For SQL Objects the idea would be to leverage tsql-t and [sql server unit tests](https://docs.microsoft.com/en-us/sql/ssdt/walkthrough-creating-and-running-a-sql-server-unit-test?view=sql-server-ver15)

![Tests](/images/Tests.PNG)


#### Developers Create a Pull Request to Merge to main Branch

Once the tests have been defined developers would like to push these 3 features into the main branch. In order to maintain sanity on the main branch , the need to have a pull request validation pipeline is essential.

In order to to do this , a PR validation pipeline needs to be created in Azure DevOps.

The basic pipeline script can be found [here](https://dev.azure.com/datalakemdw/synapsedelta/_git/synapse-delta?path=/devops/ci-test-python.yml)

![PR Validation Pipeline](/images/PRPipeline.PNG)

This pipeline uses a Pull Request trigger , which would mean every PR raised by the dev team , triggers this PR validation script which runs the Unit tests. If this test succeeds then a merge to main branch is done.

Only if the tests and the build are succesful then the merge to main branch is made possible.

A validation check is placed on the merge to main branch , which always depends on the success of the PR pipeline.

This is done by setting the branch policy in Azure DevOps on the main Branch. This can be found under the Repos section.

![PR Validation](/images/PRValidation.PNG)

Select the branch which needs to be checked before merge ( in this case master) and set the build validation to true and add the pipelines which needs to pass before the merge is succesful. This check prevents failed builds from landing on the main branch.

![PR Build Validation](/images/PRBuildValidation.PNG)
