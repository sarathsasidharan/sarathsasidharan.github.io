---
published: true
---
## How to Implement Synapse CI / CD for Data Lakes

In this post we will cover an end to end synapse CI / CD implemenation for Data Lakes for enterprise deployments. 

#### Pre-requisistes:

- [Azure Synapse Workspace](https://docs.microsoft.com/en-us/azure/synapse-analytics/quickstart-create-workspace)
- [Azure DevOps Project](https://docs.microsoft.com/en-us/azure/devops/organizations/projects/create-project?view=azure-devops&tabs=preview-page) 
- An IDE of your choice , for this blog i have used [VS Code](https://code.visualstudio.com/Download)

#### Scenario : 

Steps in this project :

1. Link Synapse Workspace Created to a Azure DevOps Repository
2. Developers Create a new branch to work on , this is done on the sandbox environment
3. Developers check in changes and raise a Pull Request 
4. Pull Request is merged into the release branch 
5. Release Branch is Deployed into the Dev Environment using Azure DevOps Pipelines 
6. Frozen Artifacts are deployed into Acceptance and Later production after integation tests have passed
