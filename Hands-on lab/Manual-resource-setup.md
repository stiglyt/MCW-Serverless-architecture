# Manual resource deployment and setup guide

This guide provides step-by-step instructions to manually provision and the configure resources created by the ARM template referenced in the before the hands-on lab guide.

April 2021

**Contents**:

- [Manual resource deployment and setup guide](#manual-resource-deployment-and-setup-guide)
  - [Task 1: Create a virtual network](#task-1-create-a-virtual-network)
  - [Task 2: Create a VPN gateway](#task-2-create-a-vpn-gateway)
  - [Task 3: Provision SQL MI](#task-3-provision-sql-mi)
  - [Task 4: Create the JumpBox VM](#task-4-create-the-jumpbox-vm)
  - [Task 5: Create SQL Server 2008 R2 virtual machine](#task-5-create-sql-server-2008-r2-virtual-machine)
  - [Task 6: Create Azure Database Migration Service](#task-6-create-azure-database-migration-service)
  - [Task 7: Provision a Web App](#task-7-provision-a-web-app)
  - [Task 8: Create an Azure Blob Storage account](#task-8-create-an-azure-blob-storage-account)
  - [Task 9: Connect to the JumpBox](#task-9-connect-to-the-jumpbox)
  - [Task 10: Install required software on the JumpBox](#task-10-install-required-software-on-the-jumpbox)
  - [Task 11: Connect to SqlServer2008 VM](#task-11-connect-to-sqlserver2008-vm)
  - [Task 12: Configure the WideWorldImporters database on the SqlServer2008 VM](#task-12-configure-the-wideworldimporters-database-on-the-sqlserver2008-vm)

> **Important**: Many Azure resources require globally unique names. Throughout these steps, you will see the word "SUFFIX" as part of resource names. You should replace this with your Microsoft alias, initials, or another value to ensure resources are uniquely named.

## Task 1: Create a virtual network

In this task, you create and configure a virtual network (VNet), which will contain your SQL managed instance, JumpBox VM, and a few other resources use throughout this hands-on lab. Once provisioned, you will associate the route table with the ManagedInstance subnet, and add a Management subnet to the VNet.

1. In the [Azure portal](https://portal.azure.com/), select the **Show portal menu** icon and then select **+Create a resource** from the menu.

   ![The Show portal menu icon is highlighted, and the portal menu is displayed. Create a resource is highlighted in the portal menu.](media/create-a-resource.png "Create a resource")

2. Enter "virtual network" into the Search the Marketplace box, and then select **Virtual Network** from the results.

   !["Virtual Network" is entered into the Search the Marketplace box. Virtual Network is selected in the results.](media/create-resource-vnet.png "Create virtual Network")

3. Select **Create** on the Virtual Network blade.

   ![The Create button is highlighted on the Virtual Network blade.](media/vnet-create.png "Create Virtual Network")

4. On the Create virtual network **Basics** tab, enter the following:

   Project details:

   - **Subscription**: Select the subscription you are using for this hands-on lab.
   - **Resource group**: Select the **hands-on-lab-SUFFIX** resource group from the list.

   Instance details:

   - **Name**: Enter `hands-on-lab-SUFFIX-vnet`
   - **Region**: Select the region you are using for resources in this hands-on lab.

   ![The values specified above are entered into the appropriate fields on the Create Virtual Network Basics tab.](media/create-virtual-network-basics-tab.png "Create virtual network Basics tab")

5. Select **Next: IP Addresses**.

6. On the **IP Addresses** tab, select **default** under subnets and edit the subnet's properties as follows:

   - **Subnet Name**: Enter `ManagedInstance`
   - **Address space**: Accept the default value. This should have a subnet mask of /24, and be within the address space indicated in the VNet's IPv4 address space, in the format **10.X.0.0/24**.
   - Select **Save**.

   ![On the Create virtual IP Addresses tab, the values specified above are entered into the appropriate fields.](media/create-virtual-network-ip-addresses-tab.png "Create virtual network IP addresses tab")

7. Select **Review + create**. The default values will be used for the remaining tabs, so they can be skipped.

8. On the **Review + create** tab, ensure the **Validation passed** message is displayed and select **Create**. It will take a few seconds for the virtual network to provision.

9. When it completes, you will get a notification in the Azure portal that the deployment succeeded. Select **Go to resource** within the notification.

   ![The Go to resource button is highlighted in the deployment succeeded notification in the Azure portal.](media/vnet-go-to-resource.png "Deployment succeeded notification")

10. On the Virtual network blade, select **Subnets** under Settings in the left-hand menu, and then select **+ Subnet** from the top menu.

    ![The Subnets item is highlighted and selected in the left-hand menu of the Virtual network blade, and + Subnet is highlighted in the top menu.](media/vnet-subnets-add.png "Add subnet")

11. On the Add subnet blade, enter the following:

    - **Name**: Enter `Management`
    - **Address range**: Accept the default value, which should be a subnet mask of /24, within the address range of your VNet.
    - **NAT gateway**: Leave set to **None**.
    - **Network security group**: Leave set to **None**.
    - **Route table**: Leave set to **None**.
    - **Service endpoints**: Leave set to **0 selected**.
    - **Subnet delegation**: Leave set to **None**.

    ![On the Add subnet blade, Management is entered into the name field, and the default values are specified for the remaining settings.](media/add-subnet-management.png "Add subnet")

12. Select **OK**.

13. Back on the **Subnets** blade, select **+ Gateway Subnet**.

    ![Subnets is selected and highlighted in the left-hand menu. On the Subnets blade, +Gateway subnet is highlighted.](media/vnet-add-gateway-subnet.png "Subnets")

14. The **Name** for gateway subnet is automatically filled in with the value `GatewaySubnet`. This value is required in order for Azure to recognize the subnet as the gateway subnet. Accept the auto-filled Address range value, and leave Route table, Service endpoints, and Subnet delegation set to their default values.

    ![The Add subnet form is displayed, with the default values.](media/vnet-add-gateway-subnet-form.png "Add subnet")

    > **Note**: The default address range creates a gateway subnet with a CIDR block of /24. This provides enough IP addresses to accommodate additional future configuration requirements.

15. Select **OK**.

## Task 2: Create a VPN gateway

In this task, you set up a Virtual Network Gateway.

1. In the [Azure portal](https://portal.azure.com/), select the **Show portal menu** icon and then select **+Create a resource** from the menu.

   ![The Show portal menu icon is highlighted, and the portal menu is displayed. Create a resource is highlighted in the portal menu.](media/create-a-resource.png "Create a resource")

2. Enter "virtual network gateway" into the Search the Marketplace box, and select **Virtual network gateway** from the results.

   !["Virtual network gateway" is entered into the Search the Marketplace box, and Virtual network gateway is highlighted in the results.](media/create-resource-virtual-network-gateway.png "Create a resource")

3. Select **Create** on the Virtual network gateway blade.

   ![The Create button is highlighted on the Virtual network gateway blade.](media/virtual-network-gateway-create.png "Virtual network gateway")

4. On the Create virtual network gateway **Basics** tab, enter the following:

   - Project details:

     - **Subscription**: Select the subscription you are using for this hands-on lab.
     - **Resource group**: This will be derived from the virtual network's resource group.

   - Instance details:

     - **Name**: Enter `hands-on-lab-SUFFIX-vnet-gateway`
     - **Region**: Select the location you are using for resources in this hands-on lab.
     - **Gateway type**: Choose **VPN**.
     - **VPN type**: Choose **Route-based**.
     - **SKU**: Select **VpnGw1**.
     - **Virtual network**: Select the **hands-on-lab-SUFFIX-vnet**.

   - Public IP address:

     - **Public IP address**: Choose **Create new**.
     - **Public IP address name**: Enter **vnet-gateway-ip**.
     - **Enable active-active mode**: Choose **Disabled**.
     - **Configure BGP ASN**: Choose **Disabled**.

   ![The values specified above are entered into the appropriate fields in the Create virtual network gateway Basics tab.](media/virtual-network-gateway-create-basics.png "Create virtual network gateway")

5. Select **Review + create**.

6. On the **Review + create** tab, ensure the _Validation passed_ message is displayed and then select **Create**.

   ![The validation passed message is displayed on the Review + create tab.](media/virtual-network-gateway-create-summary.png "Create virtual network gateway")

7. It can take up to 45 minutes for the Virtual network gateway to provision.

## Task 3: Provision SQL MI

In this task, you create an Azure SQL Managed Instance.

1. In the [Azure portal](https://portal.azure.com/), select the **Show portal menu** icon and then select **+Create a resource** from the menu.

   ![The Show portal menu icon is highlighted, and the portal menu is displayed. Create a resource is highlighted in the portal menu.](media/create-a-resource.png "Create a resource")

2. Enter "sql managed instance" into the Search the Marketplace box, and then select **Azure SQL Managed Instance** from the results.

   !["Sql managed instance" is entered into the Search the Marketplace box. Azure SQL Managed Instance is selected in the results.](media/create-resource-sql-mi.png "Create SQL Managed Instance")

3. Select **Create** on the Azure SQL Managed Instance blade.

   ![The Create button is highlighted on the Azure SQL Managed Instance blade.](media/sql-mi-create.png "Create Azure SQL Managed Instance")

4. On the Create Azure SQL Database Managed Instance Basics tab, enter the following:

   - Project details:

     - **Subscription**: Select the subscription you are using for this hands-on lab.
     - **Resource group**: Select **hands-on-lab-SUFFIX** from the list.

   - Managed Instance details:

     - **Managed instance name**: Enter `sqlmi-SUFFIX`
     - **Region**: Select the region you are using for resources in this hands-on lab.
     - **Compute + storage**: Select **Configure Managed Instance**, and on the Configure performance blade, select **Business Critical**, **Gen5**, and set the vCores to **4** and the Storage to **32**, and then select **Apply**.

     ![On the Compute + storage blade, Business Critical is selected for the service tier, the vCores are set to 4, and storage size is set to 32 GB.](media/sql-mi-configure-performance.png "Compute + storage blade")

   - Administrator account:

     - **Managed instance admin login**: Enter `sqlmiuser`
     - **Password**: Enter `Password.1234567890`

     ![On the Create SQL Managed Instance Basics tab, the values specified above are entered into the appropriate fields.](media/sql-managed-instance-basics-tab.png "Create SQL Managed Instance")

     > **Note**: If you see a message stating that Managed Instance creation is not available for the chosen subscription type, follow the instructions for [obtaining a larger quota for SQL Managed Instance](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-resource-limits#obtaining-a-larger-quota-for-sql-managed-instance).

     ![A message is displayed stating that SQL MI creation not available in the selected subscription.](media/sql-mi-creation-not-available.png "SQL MI creation not available")

5. Select **Next: Networking**, and on the **Networking** tab set the following configuration:

   - **Virtual network**: Select **hands-on-lab-SUFFIX-vnet/ManagedInstance** from the dropdown list.
   - **Prepare subnet for Managed Instance**: Select **Automatic**.
   - **Connection type**: Leave **Proxy (Default)** selected.
   - **Enable public endpoint**: Select **Disable**.

   ![On the Create SQL Managed Instance Networking tab, the configuration specified above is entered into the form.](media/sql-managed-instance-networking-tab.png "Create SQL Managed Instance")

6. Select **Next: Review + create**, and on the **Review + create** tab, review the configuration and then select **Create**.

   > **Note**: Provisioning of SQL Managed Instance can take 4+ hours if this is the first instance being deployed into a subnet. You can move on to the remaining tasks while the provisioning is in process. If the deployment process goes beyond 6 hours, you may need to submit a support ticket to request assistance in completing the setup.

## Task 4: Create the JumpBox VM

In this task, you provision a virtual machine (VM) in Azure. The VM image used has the latest version of Visual Studio Community 2019 installed.

1. In the [Azure portal](https://portal.azure.com/), select the **Show portal menu** icon and then select **+Create a resource** from the menu.

   ![The Show portal menu icon is highlighted, and the portal menu is displayed. Create a resource is highlighted in the portal menu.](media/create-a-resource.png "Create a resource")

2. Enter "visual studio 2019" into the Search the Marketplace box, and then select **Visual Studio 2019 Latest** from the results.

   !["Visual studio 2019" is entered into the Search the Marketplace box. Visual Studio 2019 Latest is selected in the search results.](./media/create-resource-visual-studio-vm.png "Create Visual Studio 2019 virtual machine")

3. On the Visual Studio 2019 blade, select **Visual Studio 2019 Community (latest release) on Windows Server 2019 (x64)** from the Select a software plan drop-down list and then select **Create**.

   ![Visual Studio 2019 Community (latest release) on Windows Server 2019 (x64) is highlighted in the Select a software plan list on the Visual Studio 2019 blade.](media/visual-studio-create.png "Visual Studio 2019")

4. On the Create a virtual machine **Basics** tab set the following configuration:

   - Project details:

     - **Subscription**: Select the subscription you are using for this hands-on lab.
     - **Resource Group**: Select the **hands-on-lab-SUFFIX** resource group from the list of existing resource groups.

   - Instance details:

     - **Virtual machine name**: Enter `JumpBox`
     - **Region**: Select the region you are using for resources in this hands-on lab.
     - **Availability options**: Select **No infrastructure redundancy required**.
     - **Image**: Leave **Visual Studio 2019 Community (latest release) on Windows Server 2019 (x64)** selected.
     - **Azure Spot instance**: Choose **No**.
     - **Size**: Accept the default size of **Standard_D4s_v3**.

   - Administrator account:

     - **Username**: Enter `sqlmiuser`
     - **Password**: Enter `Password.1234567890`

   - Inbound port rules:

     - **Public inbound ports**: Choose **Allow selected ports**.
     - **Select inbound ports**: Select **RDP (3389)** in the list.

   ![Screenshot of the Basics tab, with fields set to the previously mentioned settings.](media/lab-virtual-machine-basics-tab.png "Create a virtual machine Basics tab")

5. Select **Next: Disks** to move to the next step.

6. On the **Disks** tab, ensure the OS disk type is set to **Premium SSD**, and then select **Next: Networking**.

   ![On the Create a virtual machine Disks tab, the OS disk type is set to Standard SSD.](media/lab-virtual-machine-disks-tab.png "Create a virtual machine Disks tab")

7. On the **Networking** tab, set the following configuration:

   - **Virtual network**: Select the **hands-on-lab-SUFFIX-vnet**.
   - **Subnet**: Select the **Management** subnet.
   - **Public IP**: Leave **(new) JumpBox-ip** selected.
   - **NIC network security group**: Select **Basic**.
   - **Public inbound ports**: Leave **Allow selected ports** selected.
   - **Select inbound ports**: Leave **RDP** selected.

   ![On the Create a virtual machine Networking tab, the settings specified above are entered into the appropriate fields.](media/lab-virtual-machine-networking-tab.png "Create a virtual machine Networking tab")

8. Select **Review + create** to validate the configuration. The remaining tabs can be skipped, and default values will be used.

9. On the **Review + create** tab, ensure the Validation passed message is displayed, and then select **Create** to provision the virtual machine.

   ![The Review + create tab is displayed, with a Validation passed message.](media/lab-virtual-machine-review-create-tab.png "Create a virtual machine Review + create tab")

10. It takes approximately 15 minutes for the VM to finish provisioning. You can move on to the next task while you wait.

## Task 5: Create SQL Server 2008 R2 virtual machine

In this task, you provision another virtual machine (VM) in Azure, which will host your "on-premises" instance of SQL Server 2008 R2. The VM uses the SQL Server 2008 R2 SP3 Standard on Windows Server 2008 R2 image.

> **Note**: An older version of Windows Server is being used because SQL Server 2008 R2 is not supported on Windows Server 2016.

1. In the [Azure portal](https://portal.azure.com/), select the **Show portal menu** icon and then select **+Create a resource** from the menu.

   ![The Show portal menu icon is highlighted, and the portal menu is displayed. Create a resource is highlighted in the portal menu.](media/create-a-resource.png "Create a resource")

2. Enter "SQL Server 2008 R2 SP3 on Windows Server 2008 R2" into the Search the Marketplace box.

3. On the **SQL Server 2008 R2 SP3 on Windows Server 2008 R2** blade, select **SQL Server R2 SP3 Standard on Windows Server 2008 R2** for the software plan and then select **Create**.

   ![The SQL Server 2008 R2 SP3 on Windows Server 2008 R2 blade is displayed with the standard edition selected for the software plan. The Create button highlighted.](media/create-resource-sql-server-2008-r2.png "Create SQL Server 2008 R2 Resource")

4. On the Create a virtual machine **Basics** tab set the following configuration:

   - Project Details:

     - **Subscription**: Select the subscription you are using for this hands-on lab.
     - **Resource Group**: Select the **hands-on-lab-SUFFIX** resource group from the list of existing resource groups.

   - Instance Details:

     - **Virtual machine name**: Enter `SqlServer2008`
     - **Region**: Select the region you are using for resources in this hands-on lab.
     - **Availability options**: Select **No infrastructure redundancy required**.
     - **Image**: Leave **SQL Server 2008 R2 SP3 Standard on Windows Server 2008 R2** selected.
     - **Azure Spot instance**: Choose **No**.
     - **Size**: Accept the default size of **Standard_D4s_v3**.

   - Administrator Account:

     - **Username**: Enter `sqlmiuser`
     - **Password**: Enter `Password.1234567890`

   - Inbound Port Rules:

     - **Public inbound ports**: Choose **Allow selected ports**.
     - **Select inbound ports**: Select **RDP (3389)** in the list.

   ![Screenshot of the Basics tab, with fields set to the previously mentioned settings.](media/sql-server-2008-r2-vm-basics-tab.png "Create a virtual machine Basics tab")

5. Select **Next: Disks**, ensure the OS disk type is set to **Premium SSD**, and then select **Next: Networking**.

   ![On the Create a virtual machine Disks tab, the OS disk type is set to Standard SSD.](media/lab-virtual-machine-disks-tab.png "Create a virtual machine Disks tab")

6. On the **Networking** tab, set the following configuration:

   - **Virtual network**: Select the **hands-on-lab-vnet**.
   - **Subnet**: Select the **Management** subnet.
   - **Public IP**: Leave **(new) SqlServer2008-ip** selected.
   - **NIC network security group**: Select **Basic**.
   - **Public inbound ports**: Leave **Allow selected ports** selected.
   - **Select inbound ports**: Leave **RDP (3389)** selected.

   ![On the Create a virtual machine Networking tab, the settings specified above are entered into the appropriate fields.](media/sql-virtual-machine-networking-tab.png "Create a virtual machine Networking tab")

7. Select the **SQL Server settings** tab from the top menu.

   ![The SQL Server settings tab is highlighted in the create a virtual machine tabs list.](media/sql-server-2008-r2-vm-sql-settings-tab.png "Create a virtual machine SQL Server settings tab")

   > **Note**: The Management and Advanced tabs can be skipped, and default values will be used.

8. On the **SQL Server settings** tab, set the following:

   - Security & Networking:

     - **SQL connectivity**: Select **Public (Internet)**
     - **Port**: Set to `1433`

   - SQL Authentication:

     - **SQL Authentication**: Select **Enable**.
     - **Login name**: Enter `sqlmiuser`
     - **Password**: Enter `Password.1234567890`

     ![The previously specified values are entered into the SQL Server settings tab.](media/sql-server-2008-r2-vm-sql-settings.png "SQL Server settings")

9. Select **Review + create** to validate the configuration.

10. On the **Review + create** tab, ensure the Validation passed message is displayed, and then select **Create** to provision the virtual machine.

    ![The Review + create tab is displayed, with a Validation passed message.](media/sql-virtual-machine-review-create-tab.png "Create a virtual machine Review + create tab")

11. It takes approximately 10 minutes for the SQL VM to finish provisioning. You can move on to the next task while you wait.

## Task 6: Create Azure Database Migration Service

In this task, you provision an instance of the Azure Database Migration Service (DMS).

> **Important**: This service requires that you have registered the `Microsoft.DataMigration` resource provider within your subscription in Azure. You can find the steps to complete this in the Before the HOL guide.

1. In the [Azure portal](https://portal.azure.com/), select the **Show portal menu** icon and then select **+Create a resource** from the menu.

   ![The Show portal menu icon is highlighted, and the portal menu is displayed. Create a resource is highlighted in the portal menu.](media/create-a-resource.png "Create a resource")

2. Enter "database migration" into the Search the Marketplace box, select **Azure Database Migration Service** from the results, and select **Create** on the Azure Database Migration Service blade.

   !["Database migration" is entered into the Search the Marketplace box. Azure Database Migration Service is selected in the results.](media/create-resource-azure-database-migration-service.png "Create Azure Database Migration Service")

3. On the Create Migration Service **Basics** tab, enter the following:

   - Project details:

     - **Subscription**: Select the subscription you are using for this hands-on lab.
     - **Resource Group**: Select the **hands-on-lab-SUFFIX** resource group from the list of existing resource groups.

   - Instance details:

     - **Migration service Name**: Enter `wwi-dms`
     - **Location**: Select the location you are using for resources in this hands-on lab.
     - **Service mode**: Select **Azure**.
     - **Pricing tier**: Select **Configure tier**, choose **Premium**, and select **Apply**.

     ![The Create Migration Service Basics tab is displayed, with the values specified above entered into the appropriate fields.](media/create-migration-service-basics-tab.png "Create Migration Service")

4. Select **Next: Networking**.

5. On the **Networking** tab, select the **hands-on-lab-SUFFIX-vnet/default** virtual network by checking the box next to it in the list of existing virtual networks. This places the DMS instance into the same VNet as your LabVM and SqlServer2008 virtual machines.

   ![The hands-on-lab-vnet/default virtual network in checked and selected in the list of existing virtual networks on the Networking tab.](media/create-migration-service-networking-tab.png "Create Migration Service")

6. Select **Review + create**.

7. Select **Create**.

8. It can take 15 minutes to deploy the Azure Data Migration Service. You can move on to the next task while you wait.

## Task 7: Provision a Web App

In this task, you provision an App Service (Web app), which will be used for hosting the Wide World Importers web application.

1. In the [Azure portal](https://portal.azure.com/), select the **Show portal menu** icon and then select **+Create a resource** from the menu.

   ![The Show portal menu icon is highlighted, and the portal menu is displayed. Create a resource is highlighted in the portal menu.](media/create-a-resource.png "Create a resource")

2. In the [Azure portal](https://portal.azure.com/), select **+Create a resource**, enter "web app" into the Search the Marketplace box, select **Web App** from the results.

   ![+Create a resource is selected in the Azure navigation pane, and "web app" is entered into the Search the Marketplace box. Web App is selected in the results.](media/create-resource-web-app.png "Create Web App")

3. On the Web App blade, select **Create**.

   ![On the Web App blade, the Create button is highlighted.](media/create-web-app.png "Create Web App")

4. On the Create Web App **Basics** tab, set the following configuration:

   - Project Details:

     - **Subscription**: Select the subscription you are using for this hands-on lab.
     - **Resource Group**: Select the **hands-on-lab-SUFFIX** resource group from the list of existing resource groups.

   - Instance Details:

     - **Name**: Enter `wwi-web-SUFFIX`, to create a globally unique name.
     - **Publish**: Select **Code**.
     - **Runtime stack**: Select **.NET Core 3.1 (LTS)**.
     - **Operating System**: Select **Windows**.
     - **Region**: Select the region you are using for resources in this hands-on lab.

   - App Service Plan:

     - **Plan**: Accept the default value for creating a new App Service Plan.
     - **Sku and size**: Accept the default value of **Standard S1**.

     ![The values specified above are entered into the appropriate fields in the Create Web App Basics tab.](media/create-web-app-basics-tab.png "Create Web App Basics tab")

5. Select **Review + create**.

6. On the **Review + create** tab, select **Create**.

7. It takes a few minutes for the Web App creation to complete. You can move on to the next task while you wait.

## Task 8: Create an Azure Blob Storage account

In this task, you create an Azure Storage account.

1. In the [Azure portal](https://portal.azure.com/), select the **Show portal menu** icon and then select **+Create a resource** from the menu.

   ![The Show portal menu icon is highlighted, and the portal menu is displayed. Create a resource is highlighted in the portal menu.](media/create-a-resource.png "Create a resource")

2. Enter "storage account" into the Search the Marketplace box, select **Storage account - blob, file, table, queue** from the results, and then select **Create** on the Storage account blade.

   !["Storage account" is entered into the Search the Marketplace box. Storage account is selected in the results.](media/create-resource-storage-account.png "Create Storage account")

3. On the Create storage account blade, enter the following:

   - Project details:

     - **Subscription**: Select the subscription you are using for this hands-on lab.
     - **Resource Group**: Select the **hands-on-lab-SUFFIX** resource group from the list of existing resource groups.

   - Instance details:

     - **Storage account name**: Enter `sqlmistoreSUFFIX`
     - **Location**: Select the location you are using for resources in this hands-on lab.
     - **Performance**: Choose **Standard**.
     - **Account kind**: Select **StorageV2 (general purpose v2)**.
     - **Replication**: Select **Locally-redundant storage (LRS)**.
     - **Access tier**: Choose **Hot**.

     ![On the Create storage account blade, the values specified above are entered into the appropriate fields.](media/storage-create-account-basics-tab.png "Create storage account")

4. Select **Review + create**.

5. On the **Review + create** blade, ensure the Validate passed message is displayed and then select **Create**.

   ![On the Review + create blade, the Validation passed message is displayed at the top.](media/storage-create-account-review.png "Create storage account")

## Task 9: Connect to the JumpBox

In this task, you create an RDP connection to your JumpBox virtual machine (VM) and disable Internet Explorer Enhanced Security Configuration.

> **Note**: You do not need to wait for SQL MI to finish provisioning to complete the remaining tasks.

1. When your JumpBox VM provisioning completes, navigate to the [Azure portal](https://portal.azure.com) and select **Resource groups** from the Azure services list.

   ![Resource groups is highlighted in the Azure services list.](media/azure-services-resource-groups.png "Azure services")

2. Select the hands-on-lab-SUFFIX resource group from the list.

   ![Resource groups is selected in the Azure navigation pane, and the "hands-on-lab-SUFFIX" resource group is highlighted.](./media/resource-groups.png "Resource groups list")

3. In the list of resources for your resource group, select the JumpBox VM.

   ![The list of resources in the hands-on-lab-SUFFIX resource group are displayed, and JumpBox is highlighted.](./media/resource-group-resources-jumpbox.png "JumpBox in resource group list")

4. On your JumpBox VM blade, select **Connect** and **RDP** from the top menu.

   ![The JumpBox VM blade is displayed, with the Connect and RDP button highlighted in the top menu.](./media/connect-vm-rdp.png "Connect to JumpBox VM")

5. On the Connect with RDP blade, select **Download RDP File**, then open the downloaded RDP file.

   ![The Connect with RDP blade is displayed, and the Download RDP File button is highlighted.](./media/connect-to-virtual-machine-with-rdp.png "Connect with RDP")

6. Select **Connect** on the Remote Desktop Connection dialog.

   ![In the Remote Desktop Connection Dialog Box, the Connect button is highlighted.](./media/remote-desktop-connection.png "Remote Desktop Connection dialog")

7. Enter the following credentials when prompted, and then select **OK**:

   - **Username**: `sqlmiuser`
   - **Password**: `Password.1234567890`

   ![The credentials specified above are entered into the Enter your credentials dialog.](media/rdc-credentials.png "Enter your credentials")

8. Select **Yes** to connect, if prompted that the identity of the remote computer cannot be verified.

   ![In the Remote Desktop Connection dialog box, a warning states that the identity of the remote computer cannot be verified, and asks if you want to continue anyway. At the bottom, the Yes button is circled.](./media/remote-desktop-connection-identity-verification-jumpbox.png "Remote Desktop Connection dialog")

9. Once logged in, launch the **Server Manager**. This should start automatically, but you can access it via the Start menu if it does not.

10. Select **Local Server**, then select **On** next to **IE Enhanced Security Configuration**.

    ![Screenshot of the Server Manager. In the left pane, Local Server is selected. In the right, Properties (For LabVM) pane, the IE Enhanced Security Configuration, which is set to On, is highlighted.](./media/windows-server-manager-ie-enhanced-security-configuration.png "Server Manager")

11. In the Internet Explorer Enhanced Security Configuration dialog, select **Off** under both Administrators and Users, and then select **OK**.

    ![Screenshot of the Internet Explorer Enhanced Security Configuration dialog box, with Administrators set to Off.](./media/internet-explorer-enhanced-security-configuration-dialog.png "Internet Explorer Enhanced Security Configuration dialog box")

12. Close the Server Manager, but leave the connection to the JumpBox open for the next task.

## Task 10: Install required software on the JumpBox

In this task, you download the lab starter solution and install SQL Server Management Studio (SSMS) on the JumpBox.

1. First, download the lab starter solution from the [MCW Migrating SQL databases to Azure GitHub repo](https://github.com/microsoft/MCW-Migrating-SQL-databases-to-Azure/archive/master.zip).

2. If you receive a message that downloads are not allowed, select the Tools icon at the top right of the Internet Explorer browser window, and then select **Internet options** from the context menu.

   ![The Tools icon is highlighted in the Internet Explorer toolbar, and Internet Options is highlighted in the context menu.](media/ie-tools-context-menu.png "Internet Explorer")

3. In the **Internet Options** dialog, select **Custom level** in the Security level for this zone box.

   ![The Custom level button is highlighted in the Internet Options dialog.](media/ie-internet-options.png "Internet Options")

4. In the Security Settings - Internet Zone dialog, locate the **Downloads** settings and choose **Enable**, then select **OK**.

   ![The Downloads property is highlighted in the Security Settings dialog, and Enable is selected.](media/ie-security-settings-internet-zone.png "Security Settings")

5. Select **OK** on the Internet Options dialog, and then attempt the download again.

6. When prompted, choose to save the file and then select Open folder.

   ![The download bar is displayed in Internet Explorer, and Open folder is highlighted.](media/ie-download-open-folder.png "Internet Explorer")

7. Once it is download, extract the ZIP file to `C:\hands-on-lab`.

   ![In the Extract Compressed Zip File dialog, C:\hands-on-lab is entered into the destination field.](media/extract-compressed-zip.png "Extract Compressed Zip")

   > **Important**: Ensure you use the path above, or something similarly short. Failure to do so could result in errors opening some of the files due to a long file path.

8. Next, install SQL Server Management Studio (SSMS) on the JumpBox. Open a web browser on your JumpBox, navigate to <https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms> and then select the **Download SQL Server Management Studio (SSMS).x** link to download the latest version of SSMS.

   ![The Download SQL Server Management Studio (SSMS) link is highlighted on the page specified above.](media/download-ssms.png "Download SSMS")

   > **Note**: Versions change frequently, so if the version number you see does not match the screenshot, download and install the most recent version.

9. Run the downloaded installer.

10. On the Welcome screen, select **Install** to begin the installation.

    ![The Install button is highlighted on the SSMS installation welcome screen.](media/ssms-install.png "Install SSMS")

11. Select **Close** when the installation completes.

    ![The Close button is highlighted on the SSMS Setup Completed dialog.](media/ssms-install-close.png "Setup completed")

## Task 11: Connect to SqlServer2008 VM

In this task, you open an RDP connection to the SqlServer2008 VM, disable Internet Explorer Enhanced Security Configuration, and add a firewall rule to open port 1433 to inbound TCP traffic. You also install the Microsoft Data Migration Assistant (DMA).

1. As you did for the JumpBox, navigate to the SqlServer2008 VM blade in the Azure portal, select **Overview** from the left-hand menu, and then select **Connect** and **RDP** on the top menu.

   ![The SqlServer2008 VM blade is displayed, with the Connect button highlighted in the top menu.](./media/connect-vm-rdp.png "Connect to SqlServer2008 VM")

2. On the Connect with RDP blade, select **Download RDP File**, then open the downloaded RDP file.

3. Select **Connect** on the Remote Desktop Connection dialog.

   ![In the Remote Desktop Connection Dialog Box, the Connect button is highlighted.](./media/remote-desktop-connection-sql-2008.png "Remote Desktop Connection dialog")

4. Enter the following credentials when prompted, and then select **OK**:

   - **Username**: `sqlmiuser`
   - **Password**: `Password.1234567890`

   ![The credentials specified above are entered into the Enter your credentials dialog.](media/rdc-credentials-sql-2008.png "Enter your credentials")

5. Select **Yes** to connect, if prompted that the identity of the remote computer cannot be verified.

   ![In the Remote Desktop Connection dialog box, a warning states that the identity of the remote computer cannot be verified, and asks if you want to continue anyway. At the bottom, the Yes button is circled.](./media/remote-desktop-connection-identity-verification-sqlserver2008.png "Remote Desktop Connection dialog")

6. Once logged in, launch the **Server Manager**. This should start automatically, but you can access it via the Start menu if it does not.

7. On the **Server Manager** view, select **Configure IE ESC** under Security Information.

   ![Screenshot of the Server Manager. In the left pane, Local Server is selected. In the right, Properties (For LabVM) pane, the IE Enhanced Security Configuration, which is set to On, is highlighted.](./media/windows-server-2008-manager-ie-enhanced-security-configuration.png "Server Manager")

8. In the Internet Explorer Enhanced Security Configuration dialog, select **Off** under both Administrators and Users, and then select **OK**.

   ![Screenshot of the Internet Explorer Enhanced Security Configuration dialog box, with Administrators set to Off.](./media/2008-internet-explorer-enhanced-security-configuration-dialog.png "Internet Explorer Enhanced Security Configuration dialog box")

9. Close the Server Manager.

10. Next, you will install the Microsoft Data Migration Assistant v5.x by navigating to <https://www.microsoft.com/en-us/download/details.aspx?id=53595> in a web browser on the SqlServer2008 VM and then select the **Download** button.

    ![The Download button is highlighted on the Data Migration Assistant download page.](media/dma-download.png "Download Data Migration Assistant")

    > **Note**: Versions change frequently, so if the version number you see does not match the screenshot, download and install the most recent version.

11. Run the downloaded installer.

12. Select **Next** on each of the screens, accepting the license terms and privacy policy in the process.

13. Select **Install** on the Privacy Policy screen to begin the installation.

14. On the final screen, select **Finish** to close the installer.

    ![The Finish button is selected on the Microsoft Data Migration Assistant Setup dialog.](./media/data-migration-assistant-setup-finish.png "Run the Microsoft Data Migration Assistant")

## Task 12: Configure the WideWorldImporters database on the SqlServer2008 VM

In this task, you restore and configure the `WideWorldImporters` database on the SQL Server 2008 R2 instance.

1. On the SqlServer2008 VM, download a [backup of the WideWorldImporters database](https://raw.githubusercontent.com/microsoft/Migrating-SQL-databases-to-Azure/master/Hands-on%20lab/lab-files/Database/WideWorldImporters.bak), and save it to the `C:\` of the VM.

2. Next, open **Microsoft SQL Server Management Studio 17** (SSMS) by entering "sql server" into the search bar in the Windows Start menu and selecting **Microsoft SQL Server Management Studio 17** from the search results.

   ![SQL Server is entered into the Windows Start menu search box, and Microsoft SQL Server Management Studio 17 is highlighted in the search results.](media/start-menu-ssms-17.png "Windows start menu search")

3. In the SSMS **Connect to Server** dialog, enter **SQLSERVER2008** into the Server name box, ensure **Windows Authentication** is selected, and then select **Connect**.

   ![The SQL Server Connect to Search dialog is displayed, with SQLSERVER2008 entered into the Server name and Windows Authentication selected.](media/sql-server-connect-to-server.png "Connect to Server")

4. Once connected, right-click **Databases** under **SQLSERVER2008** in the Object Explorer, and then select **Restore Database** from the context menu.

   ![In the SSMS Object Explorer, the context menu for Databases is displayed and Restore Database is highlighted.](media/ssms-databases-restore.png "SSMS Object Explorer")

5. You will now restore the `WideWorldImporters` database using the downloaded `WideWorldImporters.bak` file. On the **General** page of the Restore Database dialog, select **Device** under Source, and then select the Browse (`...`) button to the right of the Device box.

   ![Under Source in the Restore Database dialog, Device is selected and highlighted, and the Browse button is highlighted.](media/ssms-restore-database-source.png "Restore Database source")

6. In the **Select backup devices** dialog that appears, select **Add**.

   ![In the Select backup devices dialog, the Add button is highlighted.](media/ssms-restore-database-select-devices.png "Select backup devices")

7. In the **Locate Backup File** dialog, browse to the location you saved the downloaded `WideWorldImporters.bak` file, select that file, and then select **OK**.

   ![In the Location Backup File dialog, the WideWorldImporters.bak file is selected and highlighted.](media/ssms-restore-database-locate-backup-file.png "Locate Backup File")

8. Select **OK** on the **Select backup devices** dialog. This returns you to the Restore Database dialog. The dialog now contains the information required to restore the `WideWorldImporters` database.

   ![The completed Restore Database dialog is displayed, with the WideWorldImporters database specified as the target.](media/ssms-restore-database.png "Restore Database")

9. Select **OK** to start the restore.

10. Select **OK** in the dialog when the database restore is complete.

    ![A dialog is displayed with a message that the database WideWorldImporters was restored successfully.](media/ssms-restore-database-success.png "Restored successfully")

11. Next, you execute a script in SSMS to create the `WorkshopUser` account. To create the script, open a new query window in SSMS by selecting **New Query** in the SSMS toolbar.

    ![The New Query button is highlighted in the SSMS toolbar.](media/ssms-new-query.png "SSMS Toolbar")

12. Copy and paste the SQL script below into the new query window:

    ```sql
    USE master;
    GO

    -- Create a login and user named WorkshopUser
    CREATE LOGIN WorkshopUser WITH PASSWORD = N'Password.1234567890';
    GO

    EXEC sp_addsrvrolemember
        @loginame = N'WorkshopUser',
        @rolename = N'sysadmin';
    GO

    -- Assign the user to the WideWorldImporters database
    USE WideWorldImporters;
    GO

    IF NOT EXISTS (SELECT * FROM sys.database_principals WHERE name = N'WorkshopUser')
    BEGIN
        CREATE USER [WorkshopUser] FOR LOGIN [WorkshopUser]
        EXEC sp_addrolemember N'db_datareader', N'WorkshopUser'
    END;
    GO
    ```

13. To run the script, select **Execute** from the SSMS toolbar.

    ![The Execute button is highlighted in the SSMS toolbar.](media/ssms-execute.png "SSMS Toolbar")

14. Select **New Query** from the SSMS toolbar again.

    ![The New Query button is highlighted in the SSMS toolbar.](media/ssms-new-query.png "SSMS Toolbar")

15. Next, copy and paste the SQL script below into the new query window. This script enables Service broker on the `WideWorldImporters` database.

    ```sql
    USE [WideWorldImporters];
    GO

    -- Grant the sqlmiuser ALTER database permissions
    GRANT ALTER ON DATABASE:: WideWorldImporters TO sqlmiuser;
    GO

    -- Enable Service Broker
    ALTER DATABASE WideWorldImporters
    SET ENABLE_BROKER WITH ROLLBACK IMMEDIATE;
    GO
    ```

16. To run the script, select **Execute** from the SSMS toolbar.

    ![The Execute button is highlighted in the SSMS toolbar.](media/ssms-execute.png "SSMS Toolbar")
