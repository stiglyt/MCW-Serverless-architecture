![Microsoft Cloud Workshops](https://github.com/Microsoft/MCW-Template-Cloud-Workshop/raw/main/Media/ms-cloud-workshop.png 'Microsoft Cloud Workshops')

<div class="MCWHeader1">
Serverless architecture
</div>

<div class="MCWHeader2">
Hands-on lab step-by-step
</div>

<div class="MCWHeader3">
November 2021
</div>

Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2021 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

**Contents**

<!-- TOC -->

- [Serverless architecture hands-on lab step-by-step](#serverless-architecture-hands-on-lab-step-by-step)
  - [Abstract and learning objectives](#abstract-and-learning-objectives)
  - [Overview](#overview)
  - [Solution architecture](#solution-architecture)
  - [Requirements](#requirements)
  - [Exercise 1: Develop and publish the photo processing and data export functions](#exercise-1-develop-and-publish-the-photo-processing-and-data-export-functions)
    - [Task 1: Connect to the Lab VM](#task-1-connect-to-the-lab-vm)
    - [Task 2: Open the starter solution in Visual Studio](#task-2-open-the-starter-solution-in-visual-studio)
    - [Task 3: Finish the ProcessImage function](#task-3-finish-the-processimage-function)
    - [Task 4: Publish the Function App from Visual Studio](#task-4-publish-the-function-app-from-visual-studio)
  - [Exercise 2: Create functions in the portal](#exercise-2-create-functions-in-the-portal)
    - [Task 1: Create a function to save license plate data to Azure Cosmos DB](#task-1-create-a-function-to-save-license-plate-data-to-azure-cosmos-db)
    - [Task 2: Add an Event Grid subscription to the SavePlateData function](#task-2-add-an-event-grid-subscription-to-the-saveplatedata-function)
    - [Task 3: Add an Azure Cosmos DB output to the SavePlateData function](#task-3-add-an-azure-cosmos-db-output-to-the-saveplatedata-function)
    - [Task 4: Create a function to save manual verification info to Azure Cosmos DB](#task-4-create-a-function-to-save-manual-verification-info-to-azure-cosmos-db)
    - [Task 5: Add an Event Grid subscription to the QueuePlateForManualCheckup function](#task-5-add-an-event-grid-subscription-to-the-queueplateformanualcheckup-function)
    - [Task 6: Add an Azure Cosmos DB output to the QueuePlateForManualCheckup function](#task-6-add-an-azure-cosmos-db-output-to-the-queueplateformanualcheckup-function)
  - [Exercise 3: Monitor your functions with Application Insights](#exercise-3-monitor-your-functions-with-application-insights)
    - [Task 1: Use the Live Metrics Stream to monitor functions in real-time](#task-1-use-the-live-metrics-stream-to-monitor-functions-in-real-time)
  - [Exercise 4: Explore your data in Azure Cosmos DB](#exercise-4-explore-your-data-in-azure-cosmos-db)
    - [Task 1: Use the Azure Cosmos DB Data Explorer](#task-1-use-the-azure-cosmos-db-data-explorer)
  - [Exercise 5: Create the data export workflow](#exercise-5-create-the-data-export-workflow)
    - [Task 1: Create the Logic App](#task-1-create-the-logic-app)
  - [Exercise 6: Configure continuous deployment for your Function App](#exercise-6-configure-continuous-deployment-for-your-function-app)
    - [Task 1: Add git repository to your Visual Studio solution and deploy to GitHub](#task-1-add-git-repository-to-your-visual-studio-solution-and-deploy-to-github)
    - [Task 2: Configure your Function App to use your GitHub repository for continuous deployment](#task-2-configure-your-function-app-to-use-your-github-repository-for-continuous-deployment)
    - [Task 3: Finish your ExportLicensePlates function code and push changes to GitHub to trigger deployment](#task-3-finish-your-exportlicenseplates-function-code-and-push-changes-to-github-to-trigger-deployment)
  - [Exercise 7: Rerun the workflow and verify data export](#exercise-7-rerun-the-workflow-and-verify-data-export)
    - [Task 1: Rerun Upload Images](#task-1-rerun-upload-images)
    - [Task 2: Run the Logic App](#task-2-run-the-logic-app)
    - [Task 3: View the exported CSV file](#task-3-view-the-exported-csv-file)
  - [After the hands-on lab](#after-the-hands-on-lab)
    - [Task 1: Delete the resource group in which you placed your Azure resources](#task-1-delete-the-resource-group-in-which-you-placed-your-azure-resources)
    - [Task 2: Delete the GitHub repo](#task-2-delete-the-github-repo)

<!-- /TOC -->

# Serverless architecture hands-on lab step-by-step

## Abstract and learning objectives

In this hands-on lab, you implement an end-to-end solution using a supplied sample based on Microsoft Azure Functions, Azure Cosmos DB, Azure Event Grid, and related services. The scenario will include implementing compute, storage, workflows, and monitoring using various components of Microsoft Azure. You can implement the hands-on lab on your own. However, it is highly recommended to pair up with other members at the lab to model a real-world experience and to allow each member to share their expertise for the overall solution.

At the end of the hands-on lab, you will have confidence in designing, developing, and monitoring a serverless solution that is resilient, scalable, and cost-effective.

## Overview

Contoso is rapidly expanding its toll booth management business to operate in a much larger area. As this is not their primary business, which is online payment services, they struggle with scaling up to meet the upcoming demand to extract license plate information from many new tollbooths, using photos of vehicles uploaded to cloud storage. Currently, they have a manual process where they send batches of images to a 3rd-party who manually transcodes the license plates to CSV files that they send back to Contoso to upload to their online processing system.

They want to automate this process in a way that is cost-effective and scalable. They believe serverless is the best route for them but do not have the expertise to build the solution.

## Solution architecture

Below is a diagram of the solution architecture you will build in this lab. Please study this carefully to understand the whole of the solution as you are working on the various components.

![The Solution diagram is described in the text following this diagram.](media/preferred-solution.png 'Solution diagram')

The solution begins with vehicle photos being uploaded to an **Azure Data Lake Storage Gen2** container as they are captured. An **Azure Event Grid** subscription is created against the data lake storage container. When a new blob is created, an event is triggered that calls the photo processing **Azure Function** endpoint, which in turn sends the photo to the **Computer Vision API** service to extract the license plate data. If processing is successful and the license plate number is returned. The function submits a new Event Grid event, along with the data, to an Event Grid topic with an event type called `savePlateData`. However, if the processing was unsuccessful, the function submits an Event Grid event to the topic with an event type called `queuePlateForManualCheckup`. Two separate functions are configured to trigger when new events are added to the Event Grid topic. Each filtering on a specific event type saves the relevant data to the appropriate **Azure Cosmos DB** collection for the outcome, using the Cosmos DB output binding. A **Logic App** that runs on a 15-minute interval executes an Azure Function via its HTTP trigger, responsible for obtaining new license plate data from Cosmos DB and exporting it to a new CSV file saved to Blob storage. If no new license plate records are found to export, the Logic App sends an email notification to the Customer Service department via their Office 365 subscription. **Application Insights** is used to monitor all Azure Functions in real-time as data is being processed through the serverless architecture. This real-time monitoring allows you to observe dynamic scaling first-hand and configure alerts when certain events take place. **Azure Key Vault** is used to securely store secrets, such as connection strings and access keys. Key Vault is accessed by the Function Apps through an access policy within Key Vault, assigned to each Function App's system-assigned managed identity.

## Requirements

- Microsoft Azure subscription (non-Microsoft subscription).
- Local machine or a Virtual Machine (VM) configured with (**provided as as part of this lab**):
  - Visual Studio Community 2019 or greater.
    - <https://www.visualstudio.com/vs/>
  - Azure development workload for Visual Studio.
    - <https://docs.microsoft.com/azure/azure-functions/functions-develop-vs#prerequisites>
  - .NET Core 3.1 SDK.
    - <https://www.microsoft.com/net/download/windows>
- Office 365 account. (**provided as as part of this lab**)
- GitHub account. You can create a free account at <https://github.com>.

## Exercise 1: Develop and publish the photo processing and data export functions

**Duration**: 45 minutes

Use Visual Studio and its integrated Azure Functions tooling to develop and debug the functions locally and then publish them to Azure. The starter project solution, TollBooths, contains most of the code needed. You will add in the missing code before deploying to Azure.

### Help references

|                                       |                                                                        |
| ------------------------------------- | ---------------------------------------------------------------------- |
| **Description**                       | **Link**                                                              |
| Code and test Azure Functions locally | <https://docs.microsoft.com/en-us/azure/azure-functions/functions-develop-local> |

### Task 1: Connect to the Lab VM

In this task, you use Bastion to connect to your Lab virtual machine.

1. In the [Azure portal](https://portal.azure.com), select **Resource groups** from the Azure services list.

   ![Resource groups is highlighted in the Azure services list.](media/azure-services-resource-groups.png "Azure services")

2. Select the **hands-on-lab-SUFFIX** resource group from the list.

   ![The "hands-on-lab-SUFFIX" resource group is highlighted.](media/resource-groups.png "Resource groups list")

3. In the list of resources within your resource group, select the **LabVM Virtual machine** resource.

   ![The list of resources in the hands-on-lab-SUFFIX resource group are displayed, and LabVM is highlighted.](media/resource-group-resources-labvm.png "LabVM in resource group list")

4. On your LabVM blade, select **Connect** and **Bastion** from the top menu.

   ![The LabVM blade is displayed, with the Connect button highlighted in the top menu.](media/connect-vm-rdp-new.png "Connect to Lab VM")

5. Enter the following credentials when prompted, and then select **Connect**:

   - **User name**: demouser
   - **Password**: [VM password from your teams channel]

   ![The credentials specified above are entered into the Enter your credentials dialog.](media/rdc-credentials-new.png "Enter your credentials")

### Task 2: Open the starter solution in Visual Studio

1. On the LabVM, open File Explorer and navigate to `C:\ServerlessMCW\MCW-Serverless-architecture-main\Hands-on lab\lab-files\src\TollBooth`.

    > **Note:** Ensure the files are located under `C:\ServerlessMCW\`. If the files are located under a longer root path, such as  `C:\Users\workshop\Downloads\`, you will encounter build issues in later steps: `The specified path, file name, or both are too long. The fully qualified file name must be less than 260 characters, and the directory name must be less than 248 characters`.

2. From the **TollBooth** folder opened in step 1, open the Visual Studio Solution by double-clicking the `TollBooth.sln` file.

    ![In the TollBooth folder in File Explorer, TollBooth.sln is highlighted.](media/file-explorer-toll-booth-sln.png "File Explorer")

3. If prompted about how to open the file, select **Visual Studio 2019**, and then select **OK**.

   ![Visual Studio 2019 is highlighted in the How do you want to open this file? dialog.](media/solution-file-open-with.png "Visual Studio 2019")

4. Sign in to Visual Studio using your Azure account credentials.

   ![The Sign in button is highlighted on the Visual Studio Welcome screen.](media/visual-studio-sign-in.png "Visual Studio 2019")

5. If prompted with a security warning, uncheck **Ask me for every project in this solution**, and then select **OK**.

6. Notice the solution contains the following projects:

   - `TollBooth`
   - `UploadImages`

   > **Note**: The UploadImages project is used for uploading a handful of car photos for testing the scalability of the serverless architecture.

   ![The two projects listed above are highlighted in Solution Explorer.](media/visual-studio-solution-explorer-projects.png 'Solution Explorer')

7. To validate connectivity to your Azure subscription from Visual Studio, open **Cloud Explorer** from the **View** menu and ensure that you can connect to your Azure subscription.

   ![In Cloud Explorer, the list of Azure subscriptions is shown. A single subscription is highlighted and expanded in the list.](media/vs-cloud-explorer.png 'Cloud Explorer')

   > **Note**: You may need to select the account icon and log in with your Azure account before seeing the resources below your subscription.

8. Return to the open File Explorer window and navigate back to the **src** subfolder. From there, open the **license plates** subfolder. It contains sample license plate photos used for testing out the solution. One of the images is guaranteed to fail OCR processing, which is meant to show how the workload is designed to handle such failures. The UploadImages project uses the **copyfrom** folder as a basis for the 1,000-photo upload option for testing scalability.

### Task 3: Finish the ProcessImage function

A few components within the starter project must be completed, which are marked as `TODO` in the code. The first set of `TODO` items we address are in the `ProcessImage` function. We will update the `FindLicensePlateText` class that calls the Computer Vision service and the `SendToEventGrid` class, which is responsible for sending processing results to the Event Grid topic you created earlier.

> **Note**: **Do not** update the version of any NuGet package. This solution is built to function with the NuGet package versions currently defined within. Updating these packages to newer versions could cause unexpected results.

1. From the Visual Studio **View** menu, select **Task List**.

    ![The Visual Studio Menu displays, with View and Task List selected.](media/vs-task-list-link.png 'Visual Studio Menu')

2. There, you will see a list of `TODO` tasks, where each task represents one line of code that needs to be completed.

    ![A list of TODO tasks, including their description, project, file, and line number are displayed.](media/vs-task-list.png 'TODO tasks')

    > **Note**: If the TODO ordering is out of order, select **Description** to sort it in a logical order.

3. In the Visual Studio Solution Explorer, expand the **TollBooth** project and double-click `ProcessImage.cs` to open the file.

    > Notice the Run method is decorated with the **FunctionName** attribute, which sets the name of the Azure Function to `ProcessImage`. This is triggered by HTTP requests sent to it from the Event Grid service. You tell Event Grid that you want to get these notifications at your function's URL by creating an event subscription, which you will do in a later task, in which you subscribe to blob-created events. The function's trigger watches for new blobs being added to the images container of the data lake storage account that was created by the ARM template in the Before the hands-on lab guide. The data passed to the function from the Event Grid notification includes the URL of the blob. That URL is, in turn, passed to the input binding to obtain the uploaded image from data lake storage.

    ![The ProcessImage.cs file is highlighted within the TollBooth project in the Visual Studio Solution Explorer.](media/visual-studio-solution-explorer-process-image.png "Solution Explorer")

4. In the **Task List** pane at the bottom of the Visual Studio window, double-click the `TODO 1` item, which will take you to the first `TODO` task.

    ![TODO 1 is highlighted in the Visual Studio Task List.](media/visual-studio-task-list-todo-1.png "Task List")

5. Update the code on the line below the `TODO 1` comment, using the following code:

    ```csharp
    // **TODO 1: Set the licensePlateText value by awaiting a new FindLicensePlateText.GetLicensePlate method.**
    licensePlateText = await new FindLicensePlateText(log, _client).GetLicensePlate(licensePlateImage);
    ```

6. Double-click `TODO 2` in the Task List to open the `FindLicensePlateText.cs` file.

    > This class is responsible for contacting the Computer Vision service's Read API to find and extract the license plate text from the photo using OCR. Notice that this class also shows how you can implement a resilience pattern using [Polly](https://github.com/App-vNext/Polly), an open-source .NET library that helps you handle transient errors. This is useful for ensuring that you do not overload downstream services, in this case, the Computer Vision service. This will be demonstrated later when visualizing the Function's scalability.

    ![TODO 2 is highlighted in the Visual Studio Task List.](media/visual-studio-task-list-todo-2.png "Task List")

7. The following code represents the completed task in FindLicensePlateText.cs:

    ```csharp
    // TODO 2: Populate the below two variables with the correct AppSettings properties.
    var uriBase = Environment.GetEnvironmentVariable("computerVisionApiUrl");
    var apiKey = Environment.GetEnvironmentVariable("computerVisionApiKey");
    ```

8. Double-click `TODO 3` in the Task List to open `SendToEventGrid.cs`.

    > This class is responsible for sending an Event to the Event Grid topic, including the event type and license plate data. Event listeners will use the event type to filter and act on the events they need to process. Please make a note of the event types defined here (the first parameter passed into the Send method), as they will be used later when creating new functions in the second Function App you provisioned earlier.

    ![TODO 3 is highlighted in the Visual Studio Task List.](media/visual-studio-task-list-todo-3.png "Task List")

9. Use the following code to complete `TODO 3` in `SendToEventGrid.cs`:

    ```csharp
    // TODO 3: Modify send method to include the proper eventType name value for saving plate data.
    await Send("savePlateData", "TollBooth/CustomerService", data);
    ```

10. `TODO 4` is  a few lines down from step 9 in the `else` block in `SendLicensePlateData(LicensePlateData data)`.  Use the following code to complete `TODO 4` in `SendToEventGrid.cs`:

    ```csharp
    // TODO 4: Modify send method to include the proper eventType name value for queuing plate for manual review.
    await Send("queuePlateForManualCheckup", "TollBooth/CustomerService", data);
    ```

    > **Note**: `TODOs` 5, 6, and 7 will be completed in later steps of the guide.

### Task 4: Publish the Function App from Visual Studio

In this task, you will publish the Function App from the starter project in Visual Studio to the existing Function App you provisioned in Azure.

1. Navigate to the **TollBooth** project using the Solution Explorer of Visual Studio.

2. Right-click the **TollBooth** project and select **Publish** from the context menu.

    ![In Solution Explorer, the TollBooth is selected, and within its context menu, the Publish item is selected.](media/image39.png "Solution Explorer")

3. In the Publish window, select **Azure**, then select **Next**.

    ![In the Pick a publish target window, the Azure Functions Consumption Plan is selected in the left pane. The Select Existing radio button is selected in the right pane, and the Run from package file (recommended) checkbox is unchecked. The Create Profile button is also selected.](media/vs-publish-function.png 'Publish window')

4. Select **Azure Function App (Windows)** for the specific target, then select **Next**.

    ![The specific target screen of the Publish dialog is shown with the Azure Function App (Windows) item selected and the Next button highlighted.](media/vs-publish-specific-target.png "Publish specific target")

5. In the **Publish** dialog:

    - Select your **Subscription** (1).
    - Select **Resource Group** under **View** (2).
    - In the **Function Apps** box (3), expand your **hands-on-lab-SUFFIX** resource group. Select the Function App whose name ends with **Functions**.
    - **Uncheck the `Run from package file` option** (4).

    ![In the App Service form, Resource Group displays in the View field, and in the tree-view below, the hands-on-lab-SUFFIX folder is expanded, and TollBoothFunctionApp is selected.](media/vs-publish-function2.png 'Publish window')

    > **Important**: We do not want to run from a package file because when we deploy from GitHub later on, the build process will be skipped if the Function App is configured for a zip deployment.

6. Select **Finish**.  This creates an Azure Function App publish XML file with a `.pubxml` extension.

7. Select **Publish** to start the process. Watch the Output window in Visual Studio as the Function App publishes. When it is finished, you should see a message that says, `========== Publish: 1 succeeded, 0 failed, 0 skipped ==========`.

    > **Note**: If prompted to update the version of the function on Azure, select **Yes**.

    ![The Publish button is selected.](media/vs-publish-function3.png "Publish")

8. Using a new tab or instance of your browser, navigate to the [Azure portal](https://portal.azure.com).

9. Open the **hands-on-lab-SUFFIX** resource group, then select the **TollBoothFunctions** Azure Function App, to which you just published.

10. Select **Functions** (1) in the left-hand navigation menu. You should see both functions you just published from the Visual Studio solution listed (2).

    ![In the Function Apps blade, in the left tree-view, both TollBoothFunctionApp and Functions (Read Only) are expanded. Beneath Functions (Read Only), two functions ExportLicensePlates and ProcessImage are highlighted.](media/dotnet-functions.png 'TollBoothFunctionApp blade')

11. Now, we need to add an Event Grid subscription to the ProcessImage function, so the function is triggered when new images are added to the data lake storage container.

    - Select the **ProcessImage** function.
    - Select **Integration** on the left-hand menu (1).
    - Select **Event Grid Trigger (eventGridEvent)** (2).
    - Select **Create Event Grid subscription** (3).

    ![In the TollboothFunctionApp tree-view, the ProcessImage function is selected. In the code window pane, the Add Event Grid subscription link is highlighted.](media/processimage-add-eg-sub.png 'ProcessImage function')

12. On the **Create Event Subscription** blade, specify the following configuration options:

    - **Name**: Enter a unique value, similar to **processimagesub-SUFFIX** (ensure the green check mark appears).
    - **Event Schema**: Select **Event Grid Schema**.
    - **Topic Type**: Select **Storage Accounts (Blob & GPv2)**.
    - **Subscription**: Select the subscription you are using for this hands-on lab.
    - **Resource Group**: Select the **hands-on-lab-SUFFIX** resource group from the list of existing resource groups.
    - **Resource**: Select your data lake storage account. This should be the only account listed and will start with `datalake`.
    - **System Topic Name**: Enter **processimagesubtopic-SUFFIX**.
    - **Filter to Event Types**: Select only the **Blob Created** from the event types dropdown list.
    - **Endpoint Type**: Leave `Azure Function` as the Endpoint Type.
    - **Endpoint**: Leave as `ProcessImage`.

    ![In the Create event subscription form, the fields are set to the previously defined values.](media/process-image-sub-topic.png)

13. Select **Create**.

## Exercise 2: Create functions in the portal

**Duration**: 45 minutes

In this exercise, you will create two new Azure Functions written in Node.js, using the Azure portal. These will be triggered by Event Grid and output to Azure Cosmos DB to save the results of license plate processing done by the ProcessImage function.

### Help references

|                  |          |
| ---------------- | -------- |
| **Description**  | **Link** |
| Create your first function in the Azure portal | <https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-function-app-portal> |
| Store unstructured data using Azure Functions and Azure Cosmos DB | <https://docs.microsoft.com/azure/azure-functions/functions-integrate-store-unstructured-data-cosmosdb> |

### Task 1: Create a function to save license plate data to Azure Cosmos DB

In this task, you will create a new Node.js function triggered by Event Grid that outputs successfully processed license plate data to Azure Cosmos DB.

1. Using a new tab or instance of your browser, navigate to the [Azure portal](https://portal.azure.com).

2. Open the **hands-on-lab-SUFFIX** resource group and select the Azure Function App whose name begins with **TollBoothEvents**.

3. Select **Functions** in the left-hand menu, then select **+ Create**.

    ![In the Function Apps blade, the TollBoothEvents application is selected. In the Overview tab, the + Create function button is selected.](media/functions-new.png 'TollBoothEvents blade')

4. On the **Create function** form:

   - Enter `event grid` into the **Select a template** filter box (1).
   - Select the **Azure Event Grid trigger** template (2).
   - Enter `SavePlateData` into the **New Function** name field (3).
   - Select the **Create** button (4).

    ![In the Create Function form, event grid is entered into the filter box, the Azure Event Grid trigger template is selected and highlighted, and SavePlateData is entered in the Name field and highlighted.](media/new-function-save-plate-data.png "Create Function form")

5. On the **SavePlateData** Function blade, select **Code + Test** from the left-hand menu and replace the code in the new `SavePlateData` function's `index.js` file with the following:

    ```javascript
    module.exports = function(context, eventGridEvent) {
        context.log(typeof eventGridEvent);
        context.log(eventGridEvent);

        context.bindings.outputDocument = {
            fileName: eventGridEvent.data['fileName'],
            licensePlateText: eventGridEvent.data['licensePlateText'],
            timeStamp: eventGridEvent.data['timeStamp'],
            exported: false
        };

        context.done();
    };
    ```

    ![The function code is displayed.](media/saveplatedata-code.png "SavePlateData Code + Test")

6. Select **Save**.

### Task 2: Add an Event Grid subscription to the SavePlateData function

In this task, you will add an Event Grid subscription to the SavePlateData function. This will ensure that the events sent to the Event Grid topic containing the savePlateData event type are routed to this function.

1. With the SavePlateData function open, select **Integration** in the left-hand menu, select **Event Grid Trigger (eventGridEvent)**, then select **Create Event Grid subscription**.

    ![In the SavePlateData blade code window, the Add Event Grid subscription link is selected.](media/saveplatedata-add-eg-sub.png 'SavePlateData blade')

2. On the **Create Event Subscription** blade, specify the following configuration options:

    - **Name**: Enter a unique value, similar to **saveplatedatasub-SUFFIX** (ensure the green checkmark appears).
    - **Event Schema**: Select **Event Grid Schema**.
    - **Topic Type**: Select **Event Grid Topics**.
    - **Subscription**: Select the subscription you are using for this hands-on lab.
    - **Resource Group**: Select the **hands-on-lab-SUFFIX** resource group from the list of existing resource groups.
    - **Resource**: Select your Event Grid Topic. This should be the only service listed and will start with `eventgridtopic-`.
    - **Event Types**: Select **Add Event Type** and enter `savePlateData` for the new event type value. This will ensure this Event Grid type only triggers this function.
    - **Endpoint Type**: Leave `Azure Function` as the Endpoint Type.
    - **Endpoint**: Leave as `SavePlateData`.

    ![In the Create Event Subscription blade, fields are set to the previously defined values.](media/saveplatedata-eg-sub.png "Create Event Subscription")

3. Select **Create** and then close the Edit Trigger dialog.

### Task 3: Add an Azure Cosmos DB output to the SavePlateData function

In this task, you will add an Azure Cosmos DB output binding to the SavePlateData function, enabling it to save its data to the Processed collection.

1. While still on the **SavePlateData** Integration blade, select **+ Add output** under `Outputs`.

2. In the **Create Output** blade:

   - Select the `Azure Cosmos DB` for **Binding Type** (1).
   - Beneath the Cosmos DB account connection drop down, select the **New** link (2).
   - Choose the connection whose name begins with `cosmosdb-` (3).  
   - Select **OK** (4).

    ![The Add Output link is highlighted with an arrow pointing to the highlighted binding type in the Create Output blade.](media/function-output-binding-type.png "Create Output")

3. Specify the following additional configuration options in the Create Output form:

    - **Document parameter name**: Leave set to `outputDocument`.
    - **Database name**: Enter `LicensePlates`.
    - **Collection name**: Enter `Processed`.

4. Select **OK**.

    ![Under Azure Cosmos DB output the following field values display: Document parameter name, outputDocument; Collection name, Processed; Database name, LicensePlates; Azure Cosmos DB account connection, cosmosdb_DOCUMENTDB.](media/saveplatedata-cosmos-integration.png 'Azure Cosmos DB output section')

5. Close the `SavePlateData` function.

### Task 4: Create a function to save manual verification info to Azure Cosmos DB

In this task, you will create another new function triggered by Event Grid and outputs information about photos that need to be manually verified to Azure Cosmos DB.  This is in the Azure Function App that starts with **TollBoothEvents**.

1. Select **Functions** in the left-hand menu, then select **+ Create**.

    ![In the Function Apps blade, the TollBoothEvents application is selected. In the Overview tab, the + Create button is selected.](media/functions-new.png 'TollBoothEvents blade')

2. On the **Create function** form:

   - Enter `event grid` into the **Select a template** filter box (1).
   - Select the **Azure Event Grid trigger** template (2).
   - Enter `QueuePlateForManualCheckup` into the **New Function** name field (3).
   - Select **Create** (4).

    ![In the Create function form, event grid is entered into the filter box. The Azure Event Grid trigger template is selected and highlighted, and QueuePlateForManualCheckup is entered in the Name field and highlighted.](media/new-function-manual-checkup.png "Create function form")

3. On the **QueuePlateForManualCheckup** Function blade, select **Code + Test** from the left-hand menu and replace the code in the new `QueuePlateForManualCheckup` function's `index.js` file with the following:

    ```javascript
    module.exports = async function(context, eventGridEvent) {
        context.log(typeof eventGridEvent);
        context.log(eventGridEvent);

        context.bindings.outputDocument = {
            fileName: eventGridEvent.data['fileName'],
            licensePlateText: '',
            timeStamp: eventGridEvent.data['timeStamp'],
            resolved: false
        };

        context.done();
    };
    ```

4. Select **Save**.

### Task 5: Add an Event Grid subscription to the QueuePlateForManualCheckup function

In this task, you will add an Event Grid subscription to the QueuePlateForManualCheckup function. This will ensure that the events sent to the Event Grid topic containing the queuePlateForManualCheckup event type are routed to this function.

1. With the **QueuePlateForManualCheckup** function open, select **Integration** (1) in the left-hand menu. Select **Event Grid Trigger (eventGridEvent)** (2). On the Edit Trigger form, select **Create Event Grid subscription** (3).

    ![In the QueuePlateForManualCheckup Integration blade, the Create Event Grid subscription link is selected.](media/queueplateformanualcheckup-add-eg-sub.png "QueuePlateForManualCheckup blade")

2. On the **Create Event Subscription** blade, specify the following configuration options:

    - **Name**: Enter a unique value, similar to `queueplateformanualcheckupsub-SUFFIX` (ensure the green check mark appears).
    - **Event Schema**: Select **Event Grid Schema**.
    - **Topic Type**: Select **Event Grid Topics**.
    - **Subscription**: Select the subscription you are using for this hands-on lab.
    - **Resource Group**: Select the **hands-on-lab-SUFFIX** resource group from the list of existing resource groups.
    - **Resource**: Select your Event Grid Topic. This should be the only service listed and will start with `eventgridtopic-`.
    - **Event Types**: Select **Add Event Type** and enter `queuePlateForManualCheckup` for the new event type value. This will ensure this function is only triggered by this Event Grid type.
    - **Endpoint Type**: Leave `Azure Function` as the Endpoint Type.
    - **Endpoint**: Leave as `QueuePlateForManualCheckup`.

    ![In the Create Event Subscription blade, fields are set to the previously defined values.](media/manualcheckup-eg-sub.png)

3. Select **Create** and close the Edit Trigger blade.

### Task 6: Add an Azure Cosmos DB output to the QueuePlateForManualCheckup function

In this task, you will add an Azure Cosmos DB output binding to the QueuePlateForManualCheckup function, enabling it to save its data to the NeedsManualReview collection.

1. While still on the **QueuePlateForManualCheckup** Integration blade, select **+ Add output** under **Outputs**.

2. In the **Create Output** form, select the following configuration options in the Create Output form:

    - **Binding Type**: Select `Azure Cosmos DB`.
    - **Cosmos DB account connection**: Select the **Azure Cosmos DB account connection** you created earlier.
    - **Document parameter name**: Leave set to `outputDocument`.
    - **Database name**: Enter `LicensePlates`.
    - **Collection name**: Enter `NeedsManualReview`.

3. Select **OK**.

    ![In the Azure Cosmos DB output form, the following field values display: Document parameter name, outputDocument; Collection name, NeedsManualReview; Database name, LicensePlates; Azure Cosmos DB account connection, cosmosdb-SUFFIX.](media/manual-checkup-cosmos-integration.png 'Azure Cosmos DB output form')

4. Close the **QueuePlateForManualCheckup** function.

## Exercise 3: Monitor your functions with Application Insights

**Duration**: 15 minutes

Application Insights can be integrated with Azure Function Apps to provide robust monitoring for your functions. In this exercise, you examine telemetry in the Application Insights account that you created when provisioning the Function Apps. Since you associated the Application Insights account with the Function Apps when creating them, the Application Insights telemetry key was added to the Function App configuration for you.

### Help references

|                 |          |
| --------------- | -------- |
| **Description** | **Link** |
| Monitor Azure Functions using Application Insights | <https://docs.microsoft.com/azure/azure-functions/functions-monitoring> |
| Live Metrics Stream: Monitor & Diagnose with 1-second latency | <https://docs.microsoft.com/azure/application-insights/app-insights-live-stream> |

### Task 1: Use the Live Metrics Stream to monitor functions in real-time

1. Open the **appinsights** Application Insights resource from within your lab resource group.

    ![The Application Insights instance is highlighted in the resource group.](media/resource-group-application-insights.png "Application Insights")

2. In Application Insights, select **Live Metrics Stream** under **Investigate** in the left-hand navigation menu.

    ![In the TollBoothMonitor blade, in the pane under Investigate, Live Metrics Stream is selected. ](media/live-metrics-link.png 'TollBoothMonitor blade')

3. Leave the Live Metrics Stream open and return to the starter app solution in Visual Studio on the LabVM.

4. Navigate to the **UploadImages** project using the Solution Explorer of Visual Studio. Right-click on **UploadImages** project and select **Properties**.

    ![In Solution Explorer, the UploadImages project is expanded, and Properties is selected from the right-click context menu.](media/vs-uploadimages.png 'Solution Explorer')

5. Select **Debug** in the left-hand menu, then paste the connection string for your Azure Data Lake Storage Gen2 account into the **Application arguments** text field.

   > **Note**: To obtain the connection string:
   >
   > - In the Azure portal, navigate to the **datalake{SUFFIX}** storage account.
   > - Select **Access keys** from the left menu.
   > - Copy the **Connection string** value of **key1**.
   >  

   Providing this value will ensure that the required connection string is added as an argument each time you run the application. Additionally, the combination of adding the value here and having the `.gitignore` file included in the project directory will prevent the sensitive connection string from being added to your source code repository in a later step.

    ![The Debug menu item and the command line arguments text field are highlighted.](media/vs-command-line-arguments.png "Properties - Debug")

6. Save your changes by selecting the Save icon on the Visual Studio toolbar.

7. Right-click the **UploadImages** project in the Solution Explorer, select **Debug**, then **Start New Instance** from the context menu.

    ![In Solution Explorer, the UploadImages project is selected. From the context menu, Debug then Start New Instance is selected.](media/vs-debug-uploadimages.png 'Solution Explorer')

    >**Note:** Ensure the files are located under `C:\ServerlessMCW\`. If the files are located under a longer root path, such as `C:\Users\workshop\Downloads\`, then you will encounter build issues in later steps: `The specified path, file name, or both are too long. The fully qualified file name must be less than 260 characters, and the directory name must be less than 248 characters.`

8. When the console window appears, enter **1** and press **ENTER**. This action uploads a handful of car photos to the images container of your Blob storage account.

    ![A Command prompt window displays, showing images being uploaded.](media/image69.png 'Command prompt window')

9. Switch back to your browser window with the Live Metrics Stream still open within Application Insights. You should start seeing new telemetry arrive, showing the number of servers online, the incoming request rate, CPU process amount, etc. You can select some of the sample telemetry in the list to the side to view output data.

    ![The Live Metrics Stream window displays information for the two online servers. Displaying line and point graphs including incoming requests, outgoing requests, and overall health. To the side is a list of Sample Telemetry information. ](media/image70.png 'Live Metrics Stream window')

10. Leave the Live Metrics Stream window open once again and close the console window for the image upload. Debug the UploadImages project again, then enter **2** and press **ENTER**. This will upload 1,000 new photos.

    ![The Command prompt window displays with image uploading information.](media/image71.png 'Command prompt window')

11. Switch back to the Live Metrics Stream window and observe the activity as the photos are uploaded. You can see the number of servers online, which translates to the number of Function App instances running between both Function Apps. You should also notice things such as a steady cadence for the Request Rate monitor, the Request Duration hovering below \~200ms second, and the Incoming Requests roughly matching the Outgoing Requests.

    ![In the Live Metrics Stream window, two servers are online under Incoming Requests. The Request Rate heartbeat line graph is selected, as is the Request Duration dot graph. Under Overall Health, the Process CPU heartbeat line graph is also selected, the similarities between this graph and the Request Rate graph under Incoming Requests are highlighted for comparison.](media/image72.png 'Live Metrics Stream window')

12. After this has run for a while, close the image upload console window once again, but leave the Live Metrics Stream window open.

## Exercise 4: Explore your data in Azure Cosmos DB

**Duration**: 15 minutes

In this exercise, you will use the Azure Cosmos DB Data Explorer in the portal to view saved license plate data.

### Help references

|                       |                                                           |
| --------------------- | :-------------------------------------------------------: |
| **Description**       |                         **Links**                         |
| About Azure Cosmos DB | <https://docs.microsoft.com/azure/cosmos-db/introduction> |

### Task 1: Use the Azure Cosmos DB Data Explorer

1. In the [Azure portal](https://portal.azure.com), navigate to the **hands-on-lab-SUFFIX** resource group.

   > You can get to the resource group by selecting **Resource groups** under **Azure services** on the Azure portal home page and then select the resource group from the list. If there are many resource groups in your Azure account, you can filter the list for **hands-on-lab** to reduce the resource groups listed.

2. On your resource group blade, select the **cosmosdb** Azure Cosmos DB account resource in the resource group's list of services available.

   ![The Azure Cosmos DB account resource is highlighted in the list of services in the resource group.](media/resource-group-cosmos-db-account.png "Resources")

3. On the Cosmos DB blade, select **Data Explorer** from the left-hand navigation menu.

    ![In the Data Explorer blade, Data Explorer is selected from the left menu.](media/data-explorer-link.png 'Data Explorer')

4. Expand the **LicensePlates** database and then the **Processed** collection and select **Items**. This will list each of the JSON documents added to the collection.

5. Select one of the documents to view its contents. Your functions added the first four properties. The remaining properties are standard and are assigned by Cosmos DB.

    ![In the tree-view beneath the LicensePlates Cosmos DB, the Processed collection is expanded with the Items item selected. On the Items tab, a document is selected, and its JSON data is displayed. The first four properties of the document (fileName, licensePlateText, timeStamp, and exported) are displayed along with the standard Cosmos DB properties.](media/data-explorer-processed.png 'Data Explorer')

6. Next, expand the **NeedsManualReview** collection and select **Items**.

7. Select one of the documents to view its contents. Notice that the filename is provided, as well as a property named "resolved." While out of scope for this lab, those properties can be used to provide a manual process for reviewing photos and entering license plates.

    ![In the tree-view beneath the LicensePlates Cosmos DB, the NeedsManualReview collection is expanded, and the Items item is selected. On the Items tab, a document is selected, and its JSON data is displayed. The first four properties of the document (fileName, licensePlateText, timeStamp, and resolved) are shown along with the standard Cosmos DB properties.](media/data-explorer-needsreview.png 'Data Explorer')

8. Select the ellipses (...) next to the **Processed** collection and select **New SQL Query**.

    ![In the tree-view beneath the LicensePlates Cosmos DB, the Processed collection is selected. From its right-click context menu, New SQL Query is selected.](media/data-explorer-new-sql-query.png 'Data Explorer')

9. Paste the SQL query below into the query window. This query counts the number of processed documents that have not been exported:

    ```sql
    SELECT VALUE COUNT(1) FROM c WHERE c.exported = false
    ```

10. Execute the query and observe the results. In our case, we have 669 processed documents that need to be exported.

    ![In the Query window, the previously defined SQL query displays. Under Results, the number 669 is highlighted.](media/cosmos-query-results.png 'Query 1 tab')

## Exercise 5: Create the data export workflow

**Duration**: 30 minutes

In this exercise, you create a new Logic App for your data export workflow. This Logic App will execute periodically and call your ExportLicensePlates function, then conditionally send an email if there were no records to export.

### Help references

|                 |           |
| --------------- |---------- |
| **Description** | **Links** |
| What is Azure Logic Apps? | <https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-overview> |
| Call Azure Functions from logic apps | <https://docs.microsoft.com/azure/logic-apps/logic-apps-azure-functions> |

### Task 1: Create the Logic App

1. In the [Azure portal](https://portal.azure.com), navigate to the **hands-on-lab-SUFFIX** resource group.

   > You can get to the resource group by selecting **Resource groups** under **Azure services** on the Azure portal home page and then select the resource group from the list. If there are many resource groups in your Azure account, you can filter the list for **hands-on-lab** to reduce the resource groups listed.

2. On your resource group blade, select the **logicapp** Logic App resource in the resource group's list of services available.

3. In the **Logic App Designer**, scroll through the page until you locate the _Start with a common trigger_ section. Select the **Recurrence** trigger.

    ![The Recurrence tile is selected in the Logic App Designer.](media/logic-app-designer.png 'Logic App Designer')

4. Enter **15** into the **Interval** box, and make sure Frequency is set to **Minute**. This can be set to an hour or some other interval, depending on business requirements.

5. Select **+ New step**.

    ![Under Recurrence, the Interval field is set to 15, and the + New step button is selected.](media/image83.png 'Logic App Designer Recurrence section')

6. Enter `Functions` in the filter box, then select the **Azure Functions** connector.

    ![Under Choose an action, Functions is typed in the search box. Under Connectors, Azure Functions is selected.](media/logic-app-choose-operation-functions.png 'Logic App Designer Choose an action section')

7. Select your **TollBoothFunctions** Function App.

    ![Under Azure Functions, in the search results list, Azure Functions (TollBoothFunctions) is selected.](media/logic-app-function-app-action.png 'Logic App Designer Azure Functions section')

8. Select the **ExportLicensePlates** function from the list.

    ![Under Azure Functions, under Actions (2), Azure Functions (ExportLicensePlates) is selected.](media/logic-app-select-export-function.png 'Logic App Designer Azure Functions section')

    > This function does not require any parameters that need to be sent when it gets called.

9. Select **+ New step**, then search for `condition`. Select the **Condition** Control option from the Actions search result.

    ![In the logic app designer, in the ExportLicensePlates section, the parameter field is left blank. In the Choose an action box, condition is entered as the search term, and the Condition Control item is selected from the Actions list.](media/logicapp-add-condition.png 'Logic App Designer ExportLicensePlates section')

10. For the **value** field, select the **Status code** parameter. Ensure the operator is set to **is equal to**, then enter **200** in the second value field.

    > **Note**: This evaluates the status code returned from the ExportLicensePlates function, which will return a 200 code when license plates are found and exported. Otherwise, it sends a 204 (NoContent) status code when no license plates were discovered that need to be exported. We will conditionally send an email if any response other than 200 is returned.

    ![The first Condition field displays Status code. The second, drop-down menu field displays is equal to, and the third field is set to 200.](media/logicapp-condition.png 'Condition fields')

11. We will ignore the If true condition because we don't want to perform an action if the license plates are successfully exported. Select **Add an action** within the **If false** condition block.

    ![Under the Conditions field is an If true (green checkmark) section and an if false (red x) section. In the If false section, the Add an action button is selected.](media/logicapp-condition-false-add.png 'Logic App Designer Condition fields if true/false ')

12. Enter `Send an email (V2)` in the filter box, then select the **Send an email (V2)** action for Office 365 Outlook.

    ![In the Choose an action box, send an email is entered as the search term. From the Actions list, Office 365 Outlook (end an email (V2) item is selected.](media/logicapp-send-email.png 'Office 365 Outlook Actions list')

13. Select **Sign in** and sign in with the same account you used to sign in to Azure.

    ![In the Office 365 Outlook - Send an email box, the Sign in button is selected.](media/image93.png 'Office 365 Outlook Sign in prompt')

14. In the Send an email form, provide the following values:

     - Enter your email address in the **To** box.
     - Provide a **Subject**, such as `Toll Booth license plate data export check`.
     Please enter a message into the **Body** and select the **Status code** from the ExportLicensePlates function to add it to the email body.

     >**Note:** If you receive an email with a **204** status code, all of the data has been processed. This is not an error condition. To produce a **200** status code, you will have to slow down the processing and create queued work.

     ![In the Send an email box, fields are set to the previously defined values.](media/logicapp-send-email-form.png 'Logic App Designer, Send an email fields')

15. Select **Save** in the toolbar to save your Logic App.

16. Select **Run Trigger** to execute the Logic App. You should start receiving email alerts because the license plate data is not being exported. This is because we need to finish making changes to the ExportLicensePlates function to extract the license plate data from Azure Cosmos DB, generate the CSV file, and upload it to Blob storage.

    ![The Run button is selected on the Logic Apps Designer blade toolbar.](media/logicapp-start.png 'Logic Apps Designer blade')

17. While in the Logic Apps Designer, you will see the run result of each step of your workflow. A green check mark is placed next to each step that successfully executed, showing the execution time to complete. This can be used to see how each step is working, and you can select the executed step and see the raw output.

    ![In the Logic App Designer, green check marks display next to Recurrence, ExportLicensePlates, Condition, and Send an email steps of the logic app.](media/image96.png 'Logic App Designer ')

18. The Logic App will continue to run in the background, executing every 15 minutes (or whichever interval you set) until you disable it. To disable the app, go to the **Overview** blade for the Logic App and select the **Disable** button on the taskbar.

    ![The Disable button is selected on the TollBoothLogic Logic app blade toolbar menu.](media/image97.png 'TollBoothLogic blade')

## Exercise 6: Configure continuous deployment for your Function App

**Duration**: 40 minutes

In this exercise, configure your Function App that contains the ProcessImage function for continuous deployment. You will first set up a GitHub source code repository and then set that as the Function App's deployment source.

### Help references

|                 |           |
| --------------- | ----------|
| **Description** | **Links** |
| Creating a new GitHub repository | <https://help.github.com/articles/creating-a-new-repository/> |
| Continuous deployment for Azure Functions | <https://docs.microsoft.com/azure/azure-functions/functions-continuous-deployment> |

### Task 1: Add git repository to your Visual Studio solution and deploy to GitHub

1. Return to the LabVM and in Visual Studio and select the **Git** menu item and then **Settings**.

2. In the **Options** dialog, ensure you are on the **Git Global Settings** tab under Source Control and enter your GitHub user name and email address into the `User name` and `Email` fields and then select **OK**.

    ![The Visual Studio Git Global Settings page is displayed with the user name and email fields highlighted.](media/git-global-settings.png "Git Global Settings")

3. Next, we want to set our default branch to `main`.  Select the **View** menu, then select **Terminal**.

4. In the terminal, run the following command: `git config --global init.defaultbranch main`.  This sets all default branches for new Git repositories to `main`.

5. Next, look below the Solution Explorer and select the **Git Changes** tab.

    ![The Git Changes tab is highlighted below the Solution Explorer pane in Visual Studio.](media/git-changes-tab.png "Git Changes")

6. On the Git Changes panel, select **Create Git Repository...**.

    ![In Solution Explorer, TollBooth solution is selected. From its right-click context menu, the Create Git Repository item is selected.](media/vs-create-git-repo.png 'Solution Explorer')

7. Select **Sign in...** next to Account under `Create a new GitHub repository`, then select GitHub account.

    ![The sign in button is highlighted.](media/vs-create-git-repo-sign-in.png "Create a Git repository")

8. In the web page that appears, select **Authorize github** to grant Visual Studio additional permissions to work with your GitHub account.

    > **Note**: If you did not make Microsoft Edge the default browser on the LabVM, the Authorize github button will be disabled. You will need to enter **Default apps** into the Windows Search bar and change the default web browser to Microsoft Edge.

    ![The authorize github button is highlighted.](media/vs-create-git-repo-allow.png "Allow additional permissions")

9. Sign in to your GitHub account. After a few moments, you will see a Success page appear, stating that your authorization was successful. When you see this, go back to Visual Studio.

    ![The success page is displayed.](media/vs-github-auth-successful.png "Your authorization was successful")

10. On the Create a Git repository dialog, select the browse button next to the **Local path** field to change the directory.

    ![The browse button is highlighted next to the local path for the GitHub repo.](media/create-a-git-repo-browse.png "Change local path")

11. In the Browse dialog, select the `TollBooth` folder with the `TollBooth` folder. This will select only the **TollBooth** project to add to source control and exclude the **UploadImages** project.

    ![In the Browse dialog, the TollBooth/TollBooth folder is highlighted.](media/browse-tollbooth.png "Browse")

12. Complete the form with the following information:

    - **Repository Name**: Enter `serverless-architecture-lab`.
    - **Private**: Uncheck this option.

    ![The form is completed as described.](media/vs-create-git-repo-create.png "Create a Git repository")

13. Select **Create and Push**.

14. Refresh your GitHub repository page in your browser. You should see that the project files have been added. Navigate to the **TollBooth** folder of your repo. Notice that the local.settings.json file has not been uploaded. That's because the .gitignore file of the TollBooth project explicitly excludes that file from the repository, making sure you don't accidentally share your application secrets.

    ![On the GitHub Repository webpage for serverless-architecture-lab, on the Code tab, the project files are displayed.](media/github-repo-page.png 'GitHub Repository page')

15. Close Visual Studio and reopen `TollBooth.sln` in the folder `C:\ServerlessMCW\MCW-Serverless-architecture-main\Hands-on lab\lab-files\src\TollBooth`. This is because Visual Studio closed the solution after adding the `TollBooth\TollBooth` folder as the root folder to the local repository.

### Task 2: Configure your Function App to use your GitHub repository for continuous deployment

1. In the [Azure portal](https://portal.azure.com), navigate to the **hands-on-lab-SUFFIX** resource group.

   > You can get to the resource group by selecting **Resource groups** under **Azure services** on the Azure portal home page and then select the resource group from the list. If there are many resource groups in your Azure account, you can filter the list for **hands-on-lab** to reduce the resource groups listed.

2. On your resource group blade, select the **TollBoothFunctions** Function App resource in the resource group's list of services available.

3. Select **Deployment Center** under **Deployment** in the left-hand navigation menu.

    ![The Platform features tab is displayed, under Code Deployment, Container settings is selected.](media/functionapp-menu-deployment-center-link.png 'TollBoothFunctions blade')

4. Select the **Source** drop-down list and choose **GitHub** from the list.

    ![GitHub is highlighted in the select code source drop-down list.](media/deployment-center-select-code-source.png "Select code source")

5. Select **Authorize** and enter your GitHub credentials.

    ![The Authorize button is highlighted under GitHub in the Deployment Center.](media/deployment-center-github-authorize.png "GitHub Authorize")

6. On the Authorize Azure App Service page, select **Authorize AzureAppService** and enter your password if prompted.

    ![The Authorize Azure App Service button is highlighted.](media/authorize-azure-app-service.png "Authorize Azure App Service")

7. After your account authorizes, you can configure the following to connect to your GitHub repo:

    - **Organization**: Select the GitHub account organization in which you created the repo.
    - **Repository**: Select the **serverless-architecture-lab**, or whatever name you chose for the repo.
    - **Branch**: Select **main**.

    ![The GitHub settings specified above are entered into the Settings dialog.](media/deployment-center-github-settings.png "GitHub settings")

    > **Note**: There is a current issue where the Build settings are uneditable and set to .NET version 4.0. We will change this to the proper framework version in upcoming steps.

8. Select **Save** from the top toolbar.

9. Return to the **serverless-architecture-lab** repository on the GitHub website in a web browser. From the top menu, select **Actions**.

    ![The serverless-architecture-lab repository displays with the Actions menu item highlighted.](media/githubrepo_actionsmenu.png "GitHub repository screen")

10. From beneath the **All workflows** heading, select the **Build and deploy dotnet core app to Azure Function App - TollBoothFunctions-{SUFFIX}**. Select the **main_TollBoothFunctions-{SUFFIX}** link directly below the title.

    ![The Build and deploy workflow screen is shown with the yml file link highlighted beneath the workflow title.](media/buildanddeployworkflow_landingpage.png "Workflow landing page")

    > **Note**: It is expected that the initial workflow has failed. The incorrect framework is specified in the YML document.

11. On the YML file screen, select the pencil icon to edit the document inline.

    ![The YML file screen displays with the pencil icon highlighted.](media/edit_yml_file_menu_main.png "YML file")

12. On line 14, change the **DOTNET_VERSION** value to **'3.1.x'**. Be sure not to edit the structure of this file, **ONLY** change the value. Then select **Start commit**.

    ![The YML file screen displays an editor with the DOTNET_VERSION value changed to 3.1.x, the Start commit button is highlighted.](media/yml_edit_dotnetversion_main.png "Editing a YML file")

13. In the **Commit changes** dialog, enter the comment **Changed .NET version**, then select **Commit changes**.

    ![The Commit changes dialog displays with the Changed .NET version comment and the Commit changes button highlighted.](media/yml_commit_changes_main.png "Commit changes dialog")

14. Committing the YML file update will trigger a new deployment that will succeed. You can see the status of the currently running or past workflows on the Actions tab of the repository.

    ![A successful deployment workflow displays.](media/successful_workflow_execution_main.png "Successful workflow run")

### Task 3: Finish your ExportLicensePlates function code and push changes to GitHub to trigger deployment

1. Return to the LabVM and within Visual Studio navigate to the **TollBooth** project using the Solution Explorer.

2. From the Visual Studio **View** menu, select **Task List**.

    ![Task List is selected from the Visual Studio View menu.](media/vs-task-list.png 'Visual Studio View menu')

3. In the **Task List** pane at the bottom of the Visual Studio window, double-click the `TODO 5` item, which will take you to the associated `TODO` task.

    ![TODO 5 is highlighted in the Visual Studio Task List.](media/visual-studio-task-list-todo-5.png "Task List")

4. In the **DatabaseMethods.cs** file that is opened, update the code on the line below the `TODO 5` comment, using the following code:

    ```csharp
    // TODO 5: Retrieve a List of LicensePlateDataDocument objects from the collectionLink where the exported value is false.
    licensePlates = _client.CreateDocumentQuery<LicensePlateDataDocument>(collectionLink,
            new FeedOptions() { EnableCrossPartitionQuery=true,MaxItemCount = 100 })
        .Where(l => l.exported == false)
        .ToList();
    ```

5. Next, return to the `TODO` list and double-click `TODO 6`.

    ![TODO 6 is highlighted in the Visual Studio Task List.](media/visual-studio-task-list-todo-6.png "Task List")

6. This is immediately below the `TODO 5` code you just updated. For this one, delete the line of code below the `// TODO 6` comment.

    ```csharp
    // TODO 6: Remove the line below.
    ```

7. Make sure that you deleted the following line under `TODO 6`: `licensePlates = new List<LicensePlateDataDocument>();`.

8. Save your changes to the **DatabaseMethods.cs** file.

9. Return to the `TODO` list and double-click `TODO 7`.

    ![TODO 7 is highlighted in the Visual Studio Task List.](media/visual-studio-task-list-todo-7.png "Task List")

10. In the **FileMethods.cs** file that is opened, update the code on the line below the `TODO 7` comment, using the following code:

    ```csharp
    // TODO 7: Asynchronously upload the blob from the memory stream.
    await blob.UploadFromStreamAsync(stream);
    ```

11. Save your changes to the **FileMethods.cs** file.

12. Select the **Git** menu in Visual Studio and then select **Commit or Stash...**.

    ![The Git menu is displayed.](media/vs-commit-or-stash.png "Commit or Stash")

13. Enter a commit message, then select **Commit All**.

    ![In the Team Explorer - Changes window, "Finished the ExportLicensePlates function" displays in the message box, and the Commit All button is selected.](media/vs-git-commit-all-main.png 'Team Explorer - Changes window')

14. After committing, select the **Push** button to push your changes to the GitHub repo.

    ![The Push button is highlighted.](media/vs-git-push-main.png "Push changes")

    > **Note**: You may receive a message that your local copy is behind the remote branch.
    > ![Git - Push failed dialog appears.](media/git-push-failed.png)
    >
    > If you get this, select **Pull then Push** to sync the repos and commit your changes.

15. You should see a message stating that you successfully pushed your changes to the GitHub repository.

    ![The message is displayed.](media/vs-git-push-success-main.png "Successfully pushed")

16. Go back to Deployment Center for your Function App in the portal. You should see an entry for the deployment kicked off by this last commit. Check the timestamp on the message to verify that you are looking at the latest one. **Make sure the deployment completes before continuing**.

    ![The latest deployment is displayed in the Deployment Center.](media/functionapp-dc-latest.png 'Deployment Center')

## Exercise 7: Rerun the workflow and verify data export

**Duration**: 10 minutes

With the latest code changes in place, run your Logic App and verify that the files are successfully exported.

### Task 1: Rerun Upload Images

1. In Visual Studio, right-click the **UploadImages** project in the Solution Explorer. Select **Debug**, then **Start New Instance** from the context menu.

2. When the console window appears, enter `1` and press **ENTER**. This action uploads a handful of car photos to the images container of your Blob storage account.  This should get data to trigger the ExportLicensePlates function.

### Task 2: Run the Logic App

1. Open your **hands-on-lab-SUFFIX** resource group in the Azure portal, then select your **logicapp** Logic App resource from the list.

2. From the **Overview** blade, select **Enable** (if you disabled the Logic App previously).

    ![In the TollBoothLogic Logic app blade, Overview is selected in the left menu, and the Enable enable button is selected in the right pane.](media/image113.png 'TollBoothLogic blade')

3. Now select **Run Trigger**, then select **Run** to execute your workflow immediately.

    ![In the TollBoothLogic Logic app blade, Run Trigger and Run are selected.](media/image114.png 'TollBoothLogic blade')

4. Select the **Refresh** button next to the Run Trigger button to refresh your run history. Select the latest run history item. If the expression result for the condition is **true**, then that means the CSV file should've been exported to data lake storage. Be sure to disable the Logic App, so it doesn't keep sending you emails every 15 minutes. Please note that it may take longer than expected to start running in some cases.

    ![In Logic App Designer, in the Condition section, under Inputs, true is highlighted.](media/image115.png 'Logic App Designer ')

### Task 3: View the exported CSV file

1. Open your **hands-on-lab-SUFFIX** resource group in the Azure portal, then select the **datalake** Storage account resource you provisioned to store uploaded photos and exported CSV files.

2. From the left menu of the storage account, select **Containers**, then choose the **export** container.

    ![The Containers option is selected on the left menu and the export container is highlighted from the container listing.](media/storage-containers.png 'Storage container listing')

3. You should see at least one recently uploaded CSV file. Select the filename to view its properties.

    ![In the Export blade, under Name, a .csv file is selected.](media/blob-export.png 'Export blade')

4. Select **Download** in the blob properties window.

    ![In the Blob properties blade, the Download button is selected.](media/blob-download.png 'Blob properties blade')

5. The CSV file should look similar to the following:

    ![A CSV file displays with the following columns: FileName, LicensePlateText, TimeStamp, and LicensePlateFound.](media/csv.png 'CSV file')

6. The ExportLicensePlates function updates all the records it exported by setting the exported value to true. This makes sure that only new records since the last export are included in the next one. Verify this by re-executing the script in Azure Cosmos DB that counts the number of documents in the Processed collection where exported is false. It should return 0 unless you've subsequently uploaded new photos.

## After the hands-on lab

**Duration**: 10 minutes

In this exercise, attendees will delete any Azure resources created in support of the lab.

### Task 1: Delete the resource group in which you placed your Azure resources

1. From the Portal, navigate to your **hands-on-lab-SUFFIX** resource group and select **Delete** in the toolbar at the top.

2. Confirm the deletion by re-typing the **resource group name** and selecting **Delete**.

3. If you created a different resource group for your virtual machine, be sure to delete that as well.

### Task 2: Delete the GitHub repo

[Optional] In this task, you delete the GitHub repository you created for this lab.

1. Open <https://www.github.com>, then select your profile icon and select **Your repositories**.

2. Navigate to your repo and select it.

3. Select the **Settings** tab, scroll to the bottom, select **Delete this repository**.

You should follow all steps provided _after_ attending the Hands-on lab.
