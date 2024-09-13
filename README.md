<p align="center">
    <a alt="License"
        href="https://github.com/coalesceio/salesforce/blob/salesforce_dynamic_model/LICENSE">
        <img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg" /></a>
    <a alt="Maintained?">
        <img src="https://img.shields.io/badge/Maintained%3F-yes-green.svg" /></a>
    <a alt="PRs">
        <img src="https://img.shields.io/badge/Contributions-welcome-blueviolet" /></a>
    <a alt="Fivetran Quickstart Compatible"
</p>

# Salesforce Transformation Coalesce Pipeline ([Docs](https://github.com/coalesceio/salesforce/))
# 📣 What does this Coalesce pipeline do?
- Produces modeled tables that leverage Salesforce data from [Fivetran's connector](https://fivetran.com/docs/connectors/applications/salesforce) in the format described by [this ERD](https://fivetran.com/docs/connectors/applications/salesforce#schemainformation).
- This package enables users to:
  - Understand the performance of your opportunities.
  - Drill into how the members of your sales team are performing.
  - Have a daily summary of sales activities.
  - Leverage an enhanced contact list.
  - View more details about opportunity line items.

<!--section="salesforce_transformation_model"-->
This package also generates a comprehensive data dictionary of your source and modeled Salesforce data.
You can also refer to the table below for a detailed view of all models materialized by default within this package. 

| **Model**                | **Description**                                                                                                                                |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **<span style="color:green">SALESFORCE__MANAGER_PERFORMANCE</span>**          | Each record represents a manager, enriched with data about their team's pipeline, bookings, losses, and win percentages. | 
|**<span style="color:green">SALESFORCE__SALES_SNAPSHOT</span>**            | 	A single row snapshot that provides various metrics about your sales funnel. |
| **<span style="color:green">SALESFORCE__OPPORTUNITY_ENHANCED</span>**            | Each record represents an opportunity, enriched with related data about the account and opportunity owner.|
| **<span style="color:green">SALESFORCE__DAILY_ACTIVITY</span>**            |Each record represents a daily summary of the number of sales activities, for example tasks and opportunities closed. |
| **<span style="color:green">SALESFORCE__OPPORTUNITY_LINE_ITEM_ENHANCED</span>**            |Each record represents a line item belonging to a certain opportunity, with additional product details. |
<!--section-end-->

# 🎯 How do I use the Coalesce pipeline?

## Step 1: Prerequisites
To use this Coalesce pipeline, you must have the following:

- A Fivetran Salesforce connector syncing data into your **Snowflake** destination.
- Schemas that map to Coalesce Storage Locations
- Coalesce Markeplace Packages (already included in Git repo)
  - [Base Node Types](https://docs.coalesce.io/page/package_coalesce_base-node-types) (version 1.1.2+) with - [Incremental Loading](https://docs.coalesce.io/page/package_coalesce_incremental-loading) (version 1.1.0+)
  - [Dynamic Tables](https://docs.coalesce.io/page/package_coalesce_dynamic-tables) (version 1.1.3+)

## Step 2: Import the coalesceio Salesforce repository
In Github select create a new repository and select `Import a repository`.

The URL for the Salesforce repository is [Salesforce ](https://github.com/coalesceio/salesforce.git).  This is a Public repository so no credentials are required.

Select an owner and give your repository a name.

After the import is complete you will see two branches available:

- **salesforce_dynamic_model** - This is an entire pipeline of Dynamic Tables to manage the Salesforce pipeline.
- **salesforce_incremental** - This is a pipeline built with an incremental node that loads the incremental data.

Either branch can be selected when setting up a Workspace which will be described below.

## Step 3: Set up a Project / Workspace in Coalesce
After the Git repo has been imported follow the Coalesce [documentation](https://docs.coalesce.io/docs/projects#create-a-new-project) to create a new project.  Initially, choose the option `Skip and Create` in the window for `Setup Version Control`.  We will connect to the Git repository after creating a Workspace.

Once the Project has been created select `Create Workspace`.  Enter a name and meaninful desription based on the Git branch you want to start from, either Dynamic Table or full load based.

At this point we are going to set up version control.  Select `Project Settings` and in the [Git Repository](https://docs.coalesce.io/docs/changing-a-git-repository-in-coalesce) section enter the URL of the repository you imported into your Git account as the Git Repository URL.

Save the `Project Settings`.

If you have enabled security for your Git repo, [Configure Git Account](https://docs.coalesce.io/docs/set-up-your-git-integration#add-through-the-project-dashboard).

After configuring the Git repo select `Launch` to launch the Workspace so we can attach it to a Git branch.

A Workspace can be attached to a branch by either selecting the `Git` modal or selecting `git branch` from the Workspace warning message `"Finish setting up version control for this workspace and avoid losing any work. Attach this workspace to a git branch"`.

After the `Attach Workspace to Branch` opens select the desired branch - **salesforce_dynamic_model** or **salesforce_incremental** to attach and `Attach` it.

Click on the `Git` modal, navigate to the `Branches` tab and select the [Branch Action](https://docs.coalesce.io/docs/git-branches#branch-actions) `Force Checkout` to populate the workspace with the latest Git commit.

This will overwrite any uncommitted work in the Workspace, which is what we want, so you will be required to confirm the Force Checkout by typing **FORCE** in the screen.

At this point the DAG objects should appear in your Workspace with errors.  Some workspace configuration is required to fix these errors.

## Step 3: Workspace Configuration 🛠️

In this section you will configure the following Workspace settings:
- **Build Settings** - Configure [Storage Locations](https://docs.coalesce.io/docs/storage-locations-and-storage-mappings)
- **Workspace** - Configure Settings, User Credentials, Storage Mappings and Parameters

### Build Settings
Build Settings changes are required for both the `salesforce_dynamic_model ` and `salesforce_incremental` versions of the Salesforce pipeline.

#### Storage Locations
The pipeline equires four Storage Locations be created.  
- **SOURCE** - Source location where Fivetran replicated tables are locations
- **STAGE (Default Location)** - Staging location where views are stored for staging Fivetran replicated data.  Simple things like universal filters could be applied at this layer.
- **TARGET** - The final output of the pipeline is stored in the `Target` location.

#### Environments
Environments must be configured in order to deploy pipeline to higher level environments (QA, UAT, Pre-Prod, Prod, etc.) based on how you are managing your environments.

### ⚙️Workspace Settings
The only difference between the `salesforce_dynamic_model ` and `salesforce_incremental` versions of the salesforce pipeline is that the salesforce_dynamic_table version requires some parameters to be created and set.  Other than that the configuration is the same between them.

- **Settings** - Configure the Snowflake account that Coalesce will be utilizing
- **User Credentials / OAuth Settings** - Enter the credentials required to connect to Snowflake
- **Storage Mappings** - This can be configured to use one database / schema for all Storage Locations or up to four database / schema mappings, one for each Storage Location, depending on whether or not you want to seperate Source, Staging, Intermediate and Target objects.
- **Parameters** - The Dynamic Tables in the Salesforce pipeline require two Parameters to function.  

    The first, `targetDynamicTableWarehouse` is the standard Dynamic Table Parameter described in the Dynamic Table Package documentation.  
    
    The second one, `SalesforcePipelineWarehouse` is specific to this pipeline.  It allows you to set a warehouse for the entire pipeline using a parameter instead of configuring individual nodes.  
    
    This parameter can be used or individual nodes can have their configs updated to use different warehouses.

```json
{
    "targetDynamicTableWarehouse": "DEV ENVIRONMENT",
    "SalesforcePipelineWarehouse": "<warehouse_name>"
}
```

# Running / Deploying the Pipeline
The way the Salesforce pipeline is deployed and run differs between the two versions of the pipeline.

## Dynamic Table DAG Specifics
### Pipeline LAG
The pipeline is comprised of 16 sources, 34 Dynamic Tables and 1 Date Table.  The entire Dynamic Table DAG refresh is controlled using the `Downstream` option.  The Dynamic Table which controls the `LAG` of the entire pipeline is named `PIPELINE_SYNCED`.  The option delivered with the pipeline sets a target lag = 60 minutes for the entire pipeline.

The target lag of the pipeline can be changed by changing the `Lag Specification` of the PIPELINE_SYNCED node.

Alternatively, individual nodes could be changed from Downstream to a Lag Specification.

### Missing Sources
If there are sources not available in your specific case  nodes related to those areas can be deleted from the pipeline.  This will require modification of any downstream objects that rely on sources, but should be quick to figure out utilizing Coalesce object and column level lineage capabilities.

### Executing Pipeline

After the Workspace has been configured commit any changes into Git.  If the only changes have been Build Settings and Workspace Settings then there may be nothing to commit.

At this point, you can create the pipeline. The easiest way to do this is to select `Create All` from the Graph action menu, and then execute only the Date node `INT_SALESFORCE__DATE_SPINE`.

Assuming there are no errors the Salesforce Pipeline has now been deployed into your Development environment.  From here you can deploy to higher level environments, assuming you have created environments, utilizing the standard Coalesce deployment mechanisms.

Dynamic Tables are DDL only Snowflake objects, meaning there is no `Run` component to execute.  There are no Jobs to execute either.

## Incremental Load DAG Specifics
### Pipeline Definition
The pipeline is comprised of 16 sources, 10 views and 32 tables.  

### Missing Sources
If there are sources not available in your specific case nodes related to those areas can be deleted from the pipeline.  This will require modification of any downstream objects that rely on sources, but should be quick to figure out utilizing Coalesce object and column level lineage capabilities.

### Executing Pipeline

After the Workspace has been configured commit any changes into Git.  If the only changes have been Build Settings and Workspace Settings then there may be nothing to commit.

At this point you can create the pipeline.  The easiest way to do this is select `Create All` and then you can execute with `Run All` from the Graph action menu.


From here you can deploy to higher level environments, assuming you have created environments, utilizing the standard Coalesce deployment mechanisms.

Once successfully deployed, execute the Job **SALESFORCE_LOAD** against an environment.

# 🙌 How is this pipeline maintained and can I contribute?
## Pipeline Maintenance
The Coalesce team maintaining this package _only_ maintains the latest version of the package. 

## Contributions
A small team of analytics engineers at Coalesce develops these pipelines. However, the pipelines are made better by community contributions! 

# 🏪 Are there any resources available?
- If you have questions or want to reach out for help, please refer to the [GitHub Issue](https://github.com/coalesceio/salesforce/issues) section to find the right avenue of support for you.
- If you would like to provide feedback to the Coalesce pipeline team or would like to request a new Coalesce pipeline, fill out our [Feedback Form](https:tbd).
- Any other questions about Coalesce then check out our [Help Center](https://help.coalesce.io/)
