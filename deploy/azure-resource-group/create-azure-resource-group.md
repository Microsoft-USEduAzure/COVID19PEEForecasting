# Provision Azure Resource Group

## Task: Create resource group using Azure CLI

We will create 2 Resource Groups called 

     * **COVID19BackEnd**
     * **COVID19FrontEnd**
     
     - please select the nearest data center


1. In Azure Portal, open Cloud Shell

1. Execute the following commands using Bash

    ```
    az group create -n COVID19BackEnd -l <ENTER_LOCATION_NAME>
    ```
    az group create -n COVID19FrontEnd -l <ENTER_LOCATION_NAME>

## Next task: [Create SQL Server Database](../azure-sql/provision-azure-sql.md)
