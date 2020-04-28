# Provision Azure Function

## Pre-requisite task: [Create Azure Function App](create-azure-function.md)

## Task: Deploy code Azure Function.

In this section will deploy the code to Azure. Since this a <a link ="https://docs.microsoft.com/en-us/azure/azure-functions/functions-premium-plan" target="blank">Premium Based</a> Service app, therefore to publish to Azure, this is a two step process: Deploy code to Azure using Visual Studio Code then run scripts to deploy libraries with Azure DevOps

### Task: Deploy code using Visual Studio Code

1. In your computer, open Visual Studio Code, if you don't have it installed in your computer, click <a link="https://code.visualstudio.com/download" target="blank"> here </a> to download it.

1. Navigate to **File** --> **New Window**

    ![vs code deploy](media/deploy/1.png)

1. Navigate to **File** --> **Open Folder Window**

    ![vs code deploy](media/deploy/2.png)

1. Create a new folder on your computer called COVIDFunction, select the folder

 ![vs code deploy](media/deploy/3.png)

1. Navigate to **Terminal** -->  **New Terminal**
    
    ![vs code deploy](media/deploy/4.png)

1. On the terminal window, paste the following:

    git clone https://andelcam@dev.azure.com/andelcam/COVID19%20-%20Forecasting%20Function/_git/COVID19%20-%20Forecasting%20Function
    
    ![vs code deploy](media/deploy/5.png)

1. Once the code is loaded you should see the following on the left side blade:
    
    ![vs code deploy](media/deploy/6.png)

1. Navigate to the **Azure** icon on the left side bar, enter your credentials if necessary.

    Expand the Functions blade, you will see your local project at the bottom, select CalculatePoisson.

    On the top blade of the functions list, select the deploy function icon
    
    ![vs code deploy](media/deploy/7.png)

1. Select your subscription, then select your the COVIDFunction from the dropdown box
    
    ![vs code deploy](media/deploy/8.png)

1. If there is a discrepancy on the versions, just click the **Deploy Anyway** button
    
    ![vs code deploy](media/deploy/9.png)

1. Once the deployment is completed you should see this box displayed at the bottom right corner of your screen.
    
    ![vs code deploy](media/deploy/10.png)

1. Navigate back to **Azure Portal** open **COVIDFunction** - or refresh your screen if it was already opened, you should see the function listed. Click on the **calculatePoisson** function.

    Click on the **Get function URL** link, copy the URL and paste it to NotePad
    
    ![vs code deploy](media/deploy/11.png)

### Task: Deploy libraries using Azure DevOps

1. Navigate back to  <a link="https://dev.azure.com/">Azure DevOps Portal</a> and click on the **Sign in to Azure DevOps** and enter your credentials.

    Click on the **Get function URL** link, copy the URL and paste it to NotePad
    
    ![vs code deploy](media/deploy/12.png)

1. Create a new project
    
    ![vs code deploy](media/deploy/13.png)

1. Enter the following information:

    <br>- **Project name**: COVIDFunction
    <br>- **Visibility**: Your choice

    Click on **Create**
    
    ![vs code deploy](media/deploy/14.png)

1. On the left blade navigate to **Repos** and click on **Files**:

   ![vs code deploy](media/deploy/15.png)

1. Click on the **Import Repository** option:

   ![vs code deploy](media/deploy/16.png)

1. Copy and paste the following link to the **Clone URL** text box

    https://andelcam@dev.azure.com/andelcam/COVID19%20-%20Forecasting%20Function/_git/COVID19%20-%20Forecasting%20Function

   ![vs code deploy](media/deploy/17.png)

   Click on the **Import** button

1. Once the repository is imported, you should see the files uploaded

   ![vs code deploy](media/deploy/18.png)

1. Navigate to **Pipelines**, click to **Create Pipeline**

   ![vs code deploy](media/deploy/20.png)

1. Select the COVIDFunction repository

   ![vs code deploy](media/deploy/21.png)

1. Select the Python Function App to Linux on Azure

   ![vs code deploy](media/deploy/22.png)

1. Select your subscription, select the function App name from the dropdown box. Click on the **Validate and configure**

   ![vs code deploy](media/deploy/23.png)

1. You will now see a YAML file detailing the pipeline workload, click on **Save and run**

   ![vs code deploy](media/deploy/24.png)

1. Enter your commit comments - you can leave the default message. Click on the **Save and run** button

   ![vs code deploy](media/deploy/25.png)

1. The pipeline process will begin, once the pipeline is completed your will see the following screen:

   ![vs code deploy](media/deploy/26.png)

### Task: Test the function on the portal

1. Navigate back to **Azure Portal** open **COVIDFunction**. Click on the **calculatePoisson** function. Navigate to the right blade click on test

 ![vs code deploy](media/deploy/27.png)

 1. On the request body, copy and paste the json text liste below, and click on the **Run** button

 { "x": "11", "T": "2","per_loc": "0.1","per_admit": "0.3", "per_cc": "0.25","LOS_cc": "3", "LOS_nc": "13" , "newcases":[38839,51301,67761,89505,118223,156158,206264,272448,359869,475340,627862]}

 ![vs code deploy](media/deploy/28.png)

 1. If the processing was sucessful you will see a Status: 200 OK message

 ![vs code deploy](media/deploy/29.png)

## Next task: [Deploy Website](deploy-function-app.md)