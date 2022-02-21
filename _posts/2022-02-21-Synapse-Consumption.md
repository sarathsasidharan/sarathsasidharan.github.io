---
published: false
---
## Synapse Analytics Consumption Patterns

This blog focusses on how could you expose your data using Synapse Analytics. I would first recommend you to have a quick read on [Jovan's Blog](https://techcommunity.microsoft.com/t5/azure-synapse-analytics-blog/the-best-practices-for-organizing-synapse-workspaces-and/ba-p/3002506) and [Piethein's Blog](https://towardsdatascience.com/best-practices-for-organizing-synapse-workspaces-977fe14b1fdb) on workspace design considerations.

The dicussion topic for this blog follows the choice which is made for Workspace organisation. This concerns data consumption patterns on the modern lake house architecture on Azure.

Lets assume a company named Contoso. Contoso is headquartered in Europe and has branch offices in Americas , APAC.

The company is building their lake house architecture on Azure. 

Based on the [Enterprise Scale Analytics](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/scenarios/data-management/) practises , lets assume this setup.


![Contoso Mesh](/images/contoso_data_mesh.png)


In this setup we have a data management subscription ( Blue Circle) , which is a central data platform team responsible for :

- Providing Skeleton for Infrastructure for Branches to build on
- Making sure governance is in check (non-negotiables) , using azure policies
- Providing Interfaces / Best practises for teams to leverage
- Sample DevOps Pipeleines / DataOps Pipelines for Teams to leverage and build on
- Data Project Templates which could be used by teams / branches

Assuming this team is funded by the headquarters , we have this subscription in Europe.

Other landing zones deployed in wave 1 are :

- EMEA 
- AMERICAS 
- Global Finance 
- Global HR 


### Consumption Patterns

As any other company , contoso has requirements for all 3 consumption modes :

- Batch Mode
- API Mode
- Streaming Mode

## Batch Mode

The most commonly used pattern at contoso , even today is batch mode. More than 80% of data analytics at contoso is done using the batch mode.

A typical Data warehousing pattern is adopted here too . Usual suspects of Data Loading from On premises , CRM sytems and third party DaaS (Data As a Service Providers).

Data Processing is done using an ELT approach and served in the consumption zone.

For this blog we will focus on the consumption part , we will have a seperate blog which discusses the data processing options on the Lake house pattern on Azure.

So to keep it short , assume the data is on the consumption zone ready for teams / reporting layers to consume.

![Batch Consumption](/images/batch_access.png)

Majority of data access on the lake for the initial stages is served from this pattern. Historically batch data is provided as extracts on a schedule ( monthly/weekly/daily/hourly) or on an event based trigger.

### Flow Explained

This architecture represents the data access pattern mapped to an azure data plaform. The dotted rectangles in the picture , represent core subscriptions.

These are :
- Identity 
- Management 
- Connectivity 
- Networking 

This is based on the [enterprise scale landing zone](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/) recommendation. The assumption is that ESLZ has been followed at contoso and the core cloud platfrom team has already deployed these subscrptions and the base platform is ready to be built.

The Data Platform team , builds on top of the ESLZ setup.  A dedicated data managment subscription , named Data Management Subscription Contoso has been setup.

This subscription , hosts multiple resource groups :

- network-rg ( for networking related resources)
- global-dns-rg
- governance-rg (for data governance resoruces)

This is based on the [Enterprise Scale Analytics](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/scenarios/data-management/) recommendations

Two Subscriptions are considered in this flow as an example. These are represented on the right and bottom of the data management subscription. 

APAC and Global HR (in Europe) are two subscriptions involved in this data access scenario.

Both these subscriptions are goverened by the central Data Management Contoso Subscription.

## Scenario :

Let us assume APAC HR team wants to use data stored in the global HR systems for use case A.  The Global HR team , has data in an employee table which does contain data , not only for the APAC region , but the whole of Contoso. Due to heavy regulations (GDPR) , it is crucial to only let APAC see thier own data and not of any other region , which could lead to heavy penalties and fines.

In order to get this setup working , these are the steps followed :

1. [Azure purview](https://docs.microsoft.com/en-us/azure/purview/overview#:~:text=Azure%20Purview%20is%20a%20unified%20data%20governance%20service,discovery%2C%20sensitive%20data%20classification%2C%20and%20end-to-end%20data%20lineage.) is setup in the data management subscription , inside the governance-rg resource group.
2. Global HR setups a consumption zone , where the business objects ( in this case Employee Entity) / Entities are loaded based on an SLA on data quality and avaiability . This could be a consumption folder on the storage account provisioned inside the storage-rg resource group.
3. A view [(Synapse Serverless View)](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql/create-use-views) is created on top of this entity (Employee).
4. Create a [Role Based Access Control](https://techcommunity.microsoft.com/t5/azure-synapse-analytics-blog/how-to-implement-row-level-security-in-serverless-sql-pools/ba-p/2354759) on the view which can meet the governance rule we have explained earlier ( in the scenario section).
5. Purview is going to scan the serverless view and the storage entity ( Employee) and the metadata rolls up to purview.
6. Now the metadata is available and published for other branches to find.

### Access Pattern For APAC

Once the data is registered in the catalog , the next step is for the branch to find the data they are interested in.

Steps explained :



