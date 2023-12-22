# Hand on lab: Provisioning, configuring, and getting started with development

In this lab you will create an Azure Database for PostgreSQL Flexible Server, configure it using the Azure Portal, Azure CLI and Azure REST APIs.  Once created and configured, you will then connect to it using pgAdmin to add a new 1532 dimension vector column.

## Pre-requistes

- [Azure subscription](https://azure.microsoft.com/free/)
- [Resource group](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal)

## Creating an Azure Database for Postgres Flexible Server

- Open the [Azure Portal](https://portal.azure.com/).
- Select **Create a resource (+)** in the upper-left corner of the portal or select **Create a resource** under **Azure services**.

    ![Alt text](media/01_00_create_resource.png)

- In the left side navigation, select **Databases**
- Under **Azure Database for PostgreSQL Flexible**, select **Create**

    ![Alt text](media/01_00_databases.png)

- Fill out the Basics tab with the following information:
  - Resource Group: Name of your lab resource group
  - Server name:  `PREFIX-pg-eastus-001`
  - Region: `East US`
  - PostgreSQL Version: `16`
  - Workload Type: `Production (Small/Medium-size)`
  
    ![Alt text](media/01_02_create_server_basics_00.png)

  - Under **Compute + Storage**, select **Configure Server**
  - For the size, select `Standard_D2ds_v5`

    ![Alt text](media/01_03_create_server_basics_02.png)

  - Authentication method: `PostgreSQL and Microsoft Entra authentication`
  - Admin username: `s2admin`
  - Password and confirm password: `S0lliance123`

    ![Alt text](media/01_03_create_server_basics_03.png)

- Select **Next: Networking**. On the Networking tab, you can choose how your server is reachable.
- Configure Networking options:
  - Select **Public access (allowed IP addresses)**

    ![Alt text](media/01_04_networking_01.png)

  - Add your client IP address to ensure you can connect to your new instance:
  
    ![Alt text](media/01_04_networking_02.png)

- Select **Next: Security**

    ![Alt text](media/01_05_security.png)

- Select **Next: Tags**
  
  ![Alt text](media/01_06_tags.png)

- Select **Review + create** to review your selections.

    ![Alt text](media/01_07_review_create.png)

- Select **Create** to provision the server. This operation may take a few minutes.
- In the top right of the toolbar, select the Notifications icon (a bell)

    ![Alt text](media/01_08_deployment_00.png)

- Select **Deployment in progress** link.  You can now monitor the deployment process:

    ![Alt text](media/01_08_deployment.png)

- Once deployed, select the link to navigate to your server's **Overview** page.
  - Make a note of the Server name and the Server admin login name.
  - Hover your cursor over each field, and the copy symbol appears to the right of the text.
  - Select the copy symbol as needed to copy the values for use later:
    - Subscription ID
    - Resource Group
    - Resource name
    - Server name

    ![Alt text](media/01_10_pg_overview.png)

## Configuring backup retention using Azure REST API

The Azure portal makes calls to the Azure Management API similar to how the Azure CLI and Powershell does.

- Open a new PowerShell window, run the following commands. Be sure to set the subscritionId and resourceGroup variables with the values you copied from above:

```PowerShell
$token = $(Get-AzAccessToken -ResourceUrl "https://management.azure.com/").token

$subscriptionId = "REPLACE_THIS"
$resourceGroup = "REPLACE_THIS"
$resourceName = "REPLACE_THIS"

$content = "{""properties"":{""Backup"":{""backupRetentionDays"":35}}}"

$url = "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/Microsoft.DBforPostgreSQL/flexibleServers/$($resourceName)?api-version=2023-06-01-preview"

$headers = @{
 "Authorization" = "Bearer $token"
 'content-type' = 'application/json'
}

Invoke-WebRequest -Method PATCH -Uri $url -Headers $headers -Body $content
```

## Adding a database in the portal

- Under **Settings**, select **Databases**
- In the meny, select **+Add**
- For the name, type **vector_cache**

    ![Alt text](media/01_11_pg_database_create.png)

- Select **Save**

## Configuring maintenance

- Under **Settings**, select **Maintenance**
- Select **Custom schedule**
- For the **Day of week**, select **Saturday**
- For the **Start time (UTC)**, select **23**
  
  ![Alt text](media/01_12_pg_maintenance.png)

- Select **Save**

## Configuring a server parameter  

- Under **Settings**, select **Server parameters**.
- For the **appliction_name** server parameter, type **vector_store**.
  
  ![Alt text](media/01_13_server_params_01.png)

- Select **Save**, notice an azure deployment is started.
- After the deployment is complete, switch back to the **Server parameters**.
- In the tabs, select **Static**, notice only static items are shown.
- Search for **max_connections**, then highlight the info icon. Notice the values range from 25 to 5000.
  
  ![Alt text](media/01_13_pg_server_params_static.png)

- Modify the value from **1718** to **2000**
- In the tabs, select **All**
- Search for **azure.extensions**
- Enable the **vector** extension

    ![Alt text](media/01_13_server_params_vector.png)

- Select **Save**.
- In the dialog, select **Save and Restart**

## Connecting with PG Admin

- Download and Install [pgAdmin](https://www.pgadmin.org/download/)
- Once installed, open **pgAdmin**
- Right-click the **Servers** node, select **Register->Server**
  
  ![Alt text](media/01_14_pg_admin_register.png)

- For name, type **PREFIX-pg-eastus-001**
- Select the **Connection** tab
- For the **host name/address**, paste the server name you copied from above
- For the username, type **s2admin**
- For the password, type **S0lliance123**
- Select **Save password?** to toggle it on.
- Select **Save**

## Writing your first query

Using pgAdmin, you will add a new vector column to support OpenAI embeddings.

- Expand the **PREFIX-pg-eastus-001** node
- Expand the **Databases** node
- Expand the **vector_cache->Schemas->public** nodes
- Right-click the **Tables** node, select **Create->Table**
  
  ![Alt text](media/01_15_pg_admin_create_table_menu_02.png)

- For the name, type **embeddings**
- Select **Save**
- Right-click the new table, select **Query Tool**
  
  ![Alt text](media/01_16_query_tool.png)

- Copy the following into the query tool window, this will create a typical Open AI vector column based on 1536 dimensions:

```sql
CREATE EXTENSION vector;

ALTER TABLE embeddings ADD COLUMN embedding vector(1536);
```

## Summary

In this lab, you created a new Azure Database for PostgreSQL Flexible Server instance, configured it, added a database called `vector_cache` with a `embeddinged` table that included a vector column.  

In the next set of labs, you will explore the new developer and intrastructure features of PostgreSQL 16.