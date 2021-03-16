![Microsoft Cloud Workshops](https://github.com/Microsoft/MCW-Template-Cloud-Workshop/raw/master/Media/ms-cloud-workshop.png "Microsoft Cloud Workshops")

<div class="MCWHeader1">
Serverless architecture
</div>

<div class="MCWHeader2">
Before the hands-on lab setup guide
</div>

<div class="MCWHeader3">
April 2021
</div>

Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2021 Microsoft Corporation. All rights reserved.

**Contents**

- [Serverless architecture before the hands-on lab setup guide](#serverless-architecture-before-the-hands-on-lab-setup-guide)
  - [Requirements](#requirements)
  - [Before the hands-on lab](#before-the-hands-on-lab)
    - [Task 1: Create a resource group](#task-1-create-a-resource-group)
    - [Task 2: Run ARM template to provision lab resources](#task-2-run-arm-template-to-provision-lab-resources)
    - [Task 2: Set up a development environment](#task-2-set-up-a-development-environment)
    - [Task 3: Disable IE Enhanced Security](#task-3-disable-ie-enhanced-security)
    - [Task 4: Install Microsoft Edge](#task-4-install-microsoft-edge)
    - [Task 5: Validate connectivity to Azure](#task-5-validate-connectivity-to-azure)
    - [Task 6: Download and explore the TollBooth starter solution](#task-6-download-and-explore-the-tollbooth-starter-solution)

# Serverless architecture before the hands-on lab setup guide

## Requirements

- Microsoft Azure subscription (non-Microsoft subscription)
- Local machine or a virtual machine configured with (**complete the day before the lab!**):
  - Visual Studio Community 2019 or greater
    - <https://www.visualstudio.com/vs/>
  - Azure development workload for Visual Studio 2019
    - <https://docs.microsoft.com/azure/azure-functions/functions-develop-vs#prerequisites>
  - .NET Framework 4.7 runtime (or higher) and .NET Core 3.1
    - <https://www.microsoft.com/net/download/windows>
- Office 365 account. If required, you can sign up for an Office 365 trial at:
  - <https://portal.office.com/Signup/MainSignup15.aspx?Dap=False&QuoteId=79a957e9-ad59-4d82-b787-a46955934171&ali=1>
- GitHub account. You can create a free account at <https://github.com>.

## Before the hands-on lab

**Duration**: 15 minutes

In this exercise, you set up your environment for use in the rest of the hands-on lab. You should follow all steps provided _before_ attending the hands-on lab.

> **Important**: Many Azure resources require globally unique names. Throughout these steps, the word "SUFFIX" appears as part of resource names. You should replace this with your Microsoft alias, initials, or another value to ensure uniquely named resources.

### Task 1: Create a resource group

1. In the [Azure portal](https://portal.azure.com), select **Resource groups** from the Azure services list.

   ![Resource groups is highlighted in the Azure services list.](media/azure-services-resource-groups.png "Azure services")

2. On the Resource groups blade, select **+Add**.

   ![+Add is highlighted in the toolbar on Resource groups blade.](media/resource-groups-add.png "Resource groups")

3. On the Create a resource group **Basics** tab, enter the following:

   - **Subscription**: Select the subscription you are using for this hands-on lab.
   - **Resource group**: Enter `hands-on-lab-SUFFIX` as the name of the new resource group.
   - **Region**: Select the region you are using for this hands-on lab.

   ![The values specified above are entered into the Create a resource group Basics tab.](media/create-resource-group.png "Create resource group")

4. Select **Review + Create**.

5. On the **Review + create** tab, ensure the Validation passed message is displayed and then select **Create**.

### Task 2: Run ARM template to provision lab resources

In this task, you run an Azure Resource Manager (ARM) template to create the resources required for this hands-on lab. The components are deployed inside a new virtual network (VNet) to facilitate communication between the VMs and SQL MI. The ARM template also adds inbound and outbound security rules to the network security groups associated with SQL MI and the VMs, including opening port 3389 to allow RDP connections to the JumpBox. In addition to creating resources, the ARM template also executes PowerShell scripts on each of the VMs to install software and configure the servers. The resources created by the ARM template include:

- A virtual network with three subnets, ManagedInstance, Management, and a Gateway subnet.
- A virtual network gateway associated with the Gateway subnet.
- A route table.
- Azure SQL Managed Instance (SQL MI), added to the ManagedInstance subnet.
- A JumpBox with Visual Studio 2019 Community Edition and SQL Server Management Studio (SSMS installed, added to the Management subnet).
- A SQL Server 2008 R2 VM with the Data Migration Assistant (DMA) installed, added to the Management subnet.
- Azure Database Migration Service (DMS).
- Azure App Service Plan and App Service (Web App).
- Azure Blob Storage account.

> **Note**: You can review the steps to manually provision and configure the lab resources in the [Manual resource setup guide](./Manual-resource-setup.md).

1. In the [Azure portal](https://portal.azure.com/), select the **Show portal menu** icon and then select **+Create a resource** from the menu.

   ![The Show portal menu icon is highlighted, and the portal menu is displayed. Create a resource is highlighted in the portal menu.](media/create-a-resource.png "Create a resource")

2. Before running the ARM template, it is beneficial to quickly verify that you can provision SQL MI in your subscription. In the [Azure portal](https://portal.azure.com), select **+Create a resource**, enter "sql managed instance" into the Search the Marketplace box, and then select **Azure SQL Managed Instance** from the results.

   ![+Create a resource is selected in the Azure navigation pane, and "sql managed instance" is entered into the Search the Marketplace box. Azure SQL Managed Instance is selected in the results.](media/create-resource-sql-mi.png "Create SQL Managed Instance")

3. Select **Create** on the Azure SQL Managed Instance blade.

   ![The Create button is highlighted on the Azure SQL Managed Instance blade.](media/sql-mi-create.png "Create Azure SQL Managed Instance")

4. On the SQL managed instance blade, look for a message stating that "Managed instance creation is not available for the chosen subscription type...", which will be displayed near the bottom of the SQL managed instance blade.

   ![A message is displayed stating that SQL MI creation not available in the selected subscription.](media/sql-mi-creation-not-available.png "SQL MI creation not available")

   > **Note**: If you see the message stating that Managed Instance creation is not available for the chosen subscription type, follow the instructions for [obtaining a larger quota for SQL Managed Instance](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-resource-limits#obtaining-a-larger-quota-for-sql-managed-instance) before proceeding with the following steps.

5. You are now ready to begin the ARM template deployment. To open a custom deployment screen in the Azure portal, select the Deploy to Azure button below:

   <a href ="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2FMCW-Migrating-SQL-databases-to-Azure%2Fmaster%2FHands-on%20lab%2Flab-files%2FARM-template%2Fazure-deploy.json" target="_blank" title="Deploy to Azure">
   <img src="http://azuredeploy.net/deploybutton.png"/>
   </a>

   > **Note**: Running the ARM template occasionally results in a `ResourceDeploymentFailure` error, with a code of `VnetSubnetConflictedWithIntendedPolicy`. This error is not caused by an issue with the ARM template and appears to be the result of backend resource deployment issues in Azure. At this time, the workaround is first to try the deployment in a different region. If that does not work, try going through the [Manual resource setup guide](./Manual-resource-setup.md) to create the SQL MI database.

6. On the custom deployment screen in the Azure portal, enter the following:

   - **Subscription**: Select the subscription you are using for this hands-on lab.
   - **Resource group**: Select the hands-on-lab-SUFFIX resource group from the dropdown list.
   - **Region**: Select the region you used for the hands-on-lab-SUFFIX resource group.
   - **Managed Instance Name**: Accept the default value, **sqlmi**. The actual name must be globally unique, so a unique string is generated from your Resource Group and appended to the name during provisioning.
   - **Admin Username**: Accept the default value, **sqlmiuser**.
   - **Admin Password**: Accept the default value, **Password.1234567890**.
   - **V Cores**: Accept the default value, **4**.
   - **Storage Size in GB**: Accept the default value, **32**.

   ![The Custom deployment blade is displayed, and the information above is entered on the Custom deployment blade.](media/azure-custom-deployment.png "Custom deployment blade")

7. Select **Review + create** to review the custom deployment.

8. On the Review + create blade, ensure the _Validation passed_ message is displayed and then select **Create** to begin the custom deployment.

   ![On the Review + create blade for the custom deployment, the Validation passed message is highlighted, and the Create button is highlighted.](media/azure-custom-deployment-review-create.png "Review + create custom deployment")

   > **Note**: The deployment of the custom ARM template can take over 4 hours due to the inclusion of SQL MI. However, the deployment of most of the resources completes within a few minutes. The JumpBox and SQL Server 2008 R2 VMs should finish in about 15 minutes.

9. You can monitor the deployment's progress by navigating to the hands-on-lab-SUFFIX resource group in the Azure portal and then selecting **Deployments** from the left-hand menu. The deployment is named **Microsoft.Template**. Select that to view the progress of each item in the template.

   ![The Deployments menu item is selected in the left-hand menu of the hands-on-lab-SUFFIX resource group and the Microsoft.Template deployment is highlighted.](media/resource-group-deployments.png "Resource group deployments")

> Check back in a few hours to monitor the progress of your SQL MI provisioning. If the provisioning goes on for longer than 7 hours, you may need to issue a support ticket in the Azure portal to request the provisioning process be unblocked by Microsoft support.

You should follow all steps provided _before_ attending the Hands-on lab.











### Task 2: Set up a development environment

If you do not have a machine with Visual Studio Community 2019 (or greater) and the Azure development workload, complete this task.

1. Create a virtual machine (VM) in Azure using the Visual Studio Community 2019 on Windows Server 2019 (x64) image. A Windows 10 image will work as well. **Note:** Your Azure subscription must include MSDN offers to create a VM with Visual Studio pre-loaded.

   ![In Azure Portal, in the search field, Visual Studio Community 2019 (latest release) on Windows Server 2019 (x64) is selected.](media/select-vs2019-image.png 'Azure Portal')

   - Select **+ Create a resource**.

   - Type **Visual Studio 2019 Latest**.

   - Select the **Visual Studio Community 2019 (latest) on Windows Server 2019 (x64)**.

   - Select **Create**.

   - Select your subscription and recently created resource group.

   - For Virtual machine name, type **MainVM**, or a different name that is unique.

   - Leave availability option as **No infrastructure redundancy required**.

   - Ensure the image is **Visual Studio Community 2019 (latest) on Windows Server 2019 (x64)**.

   - Select your VM size.

   > **Note**: It is highly recommended to use a D4s or DS2_v2 instance size for this VM.

   - For username, type **demouser**

   - For password, type **Password.1!!**

   - Select **Allow selected ports**.

   - For the inbound ports, select **RDP (3389)**.

   - Select **Review + create**.

   - Select **Create**.

### Task 3: Disable IE Enhanced Security

> **Note**: Sometimes this image has IE ESC disabled. Sometimes it does not.

1. Login to the newly created VM using RDP and the username and password you supplied earlier.

2. After the VM loads, the Server Manager should open.

3. Select **Local Server**.

   ![Local Server is selected from the Server Manager menu.](media/image5.png 'Server Manager menu')

4. On the side of the pane, for **IE Enhanced Security Configuration**, if it displays **On**, select it.

   ![The IE Enhanced Security Configuration setting is set to On. The On item is selected.](media/image6.png 'IE Enhanced Security Configuration')

   - Change to **Off** for Administrators and select **OK**.

   ![In the Internet Explorer Enhanced Security Configuration dialog box, under Administrators, the Off button is selected.](media/image7.png 'Internet Explorer Enhanced Security Configuration dialog box')

### Task 4: Install Microsoft Edge

> **Note**: Some aspects of this lab require the use of the new Microsoft Edge (Chromium edition) browser. You may find yourself blocked if using Internet Explorer later in the lab.

1. Launch Internet Explorer and download [Microsoft Edge](https://www.microsoft.com/edge).

2. Follow the setup instructions and make sure you can run Edge to navigate to any webpage.

> **Note**: Edge is needed for one of the labs as Internet Explorer is not supported for some specific activities.

### Task 5: Validate connectivity to Azure

1. From within the virtual machine, launch Visual Studio (select **Continue without code** link) and validate that you can log in with your Microsoft Account when prompted.

2. To validate connectivity to your Azure subscription, open **Cloud Explorer** from the **View** menu, and ensure that you can connect to your Azure subscription.

   ![In Cloud Explorer, the list of Azure subscriptions is shown. A single subscription is highlighted and expanded in the list.](media/vs-cloud-explorer.png 'Cloud Explorer')

### Task 6: Download and explore the TollBooth starter solution

1. From your LabVM, download the starter files by downloading a .zip copy of the Serverless architecture MCW GitHub repo.

2. In a web browser, navigate to the [MCW Serverless architecture repo](https://github.com/Microsoft/MCW-Serverless-architecture).

3. On the repo page, select **Clone or download**, then select **Download ZIP**.

   ![On the GitHub Repository web page, the Clone or Download drop down is expanded with the Download ZIP button selected.](media/github-download-repo.png)

4. Unzip the contents to the folder **C:\\ServerlessMCW\\**

   ![On the Extract Compressed (Zipped) Folders dialog window, the extraction path is highlighted in the Files will be extracted to this folder field.](media/zip-extract.png 'Extract Compressed Folders')

5. Navigate to `C:\ServerlessMCW\MCW-Serverless-architecture-master\Hands-on lab\starter`

6. From the **TollBooth** folder, open the Visual Studio Solution file: **TollBooth.sln**. Notice the solution contains the following projects:

   - TollBooth
   - UploadImages
   
   > **Note**: The UploadImages project is used for uploading a handful of car photos for testing scalability of the serverless architecture.

7. Switch to windows explorer, navigate back to the **starter** subfolder and open the **license plates** subfolder. It contains sample license plate photos used for testing out the solution. One of the photos is guaranteed to fail OCR processing, which is meant to show how the workload is designed to handle such failures. The **copyfrom** folder is used by the UploadImages project as a basis for the 1,000 photo upload option for testing scalability.

You should follow all steps provided _before_ performing the Hands-on lab.
