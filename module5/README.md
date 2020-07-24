Transferring SMB data using AWS DataSync
======================
Copyright Amazon Web Services, Inc. and its affiliates. All rights reserved.This sample code is made available under the MIT-0 license. See the LICENSE file.

Errors or corrections? Contact akbariw@amazon.com.

-------------------------------------------------------------------------------------
**Summary**
-------------------
AWS DataSync is a data transfer service that makes it easy for you to simplify & accelerate data migration between on-premises storage and Amazon S3, Amazon Elastic File System, or Amazon FSx for Windows File server. AWS DataSync automatically handles many of the heavy lifting tasks related to data transfers that can slow down and complicate migrations such as building & managing complex scripts that handle metadata preservation, data integrity validation, enabling parallel transfers and network optimization.

<br/><br/>

**OVERVIEW**
-------------------
In this module we are going to create a local windows SMB share on the Windows EC2 instance that will act as your source SMB share data, and also create a target Amazon FSx for Windows File Server, file share. Then deploy and use AWS DataSync to migrate the source SMB data to the target FSx for Windows File Server, file share.

<br/><br/>

**Duration**
-------------------
It will take approximately 20 minutes to complete this section.

<br/><br/>

**Create local Windows SMB Share**
-----------------------------

1. Open this link using Internet Explorer (https://view.highspot.com/viewer/5f1a645ba2e3a938135fa3f6), and then click on the **download** button at the top left of the screen, you will need this workshop file during in this module    
2. While your are logged within your Windows EC2 instance for the workshop, click on the Windows icon in the bottom left hand corner and type **C:\\** and press Enter
2. Right click on the **Tools** folder and select **Properties**
3. Click on the **Sharing** tab, and then click on **Advanced Sharing**
4. Select the box to **Share this folder** and select **OK** and close
5. Open a Windows command prompt and type in **hostname** and press Enter. Copy down the value shown for your server's hostname in your the workshop notepad file you downloaded, as you will need this later



<br/><br/>

**Create target Amazon FSx for Windows File Server, file share**
-----------------------------

We are now going to create a target SMB share on the Amazon FSx for Windows File Server instance

1. Within your Windows EC2 instance, click on the Windows START icon in the bottom left hand corner and type **fsmgmt.msc** and press Enter
2. Click on **Action** and select **Connect to another computer** 
3. Enter in your Amazon FSx for Windows File Server DNS name (e.g. amznfsxnzvcely3.example.com) and select OK
4. Context/right-click the **Shares** folder and click **New Share** and select Next
5. Click Browse
6. Select d$.
7. Click Make New Folder.
8. Name the new folder **datasync**
9. Select OK
10. Complete the Create A Shared Folder Wizard, and at the **Shared folder permissions** screen, select **Custom permissions** and then click on **Custom** and enable **Everyone** with **Full Control**


<br/><br/>

**Deploy AWS DataSync agent**
-----------------------------

We are going to deploy the AWS DataSync agent within your AWS VPC as an EC2 instance. The AWS DataSync agent will then read directly from your Windows SMB share and transfer the data to your **Amazon FSx for Windows File Server, file share**


- Within your Windows EC2 instance, click on the Windows START icon in the bottom left hand corner and type **Powershell** and select **Windows Powershell**

Then copy and paste the below into the powershell window as a single line to download and install Google Chrome 

    $Path = $env:TEMP; $Installer = "chrome_installer.exe"; Invoke-WebRequest "http://dl.google.com/chrome/install/375.126/chrome_installer.exe" -OutFile $Path\$Installer; Start-Process -FilePath $Path\$Installer -Args "/silent /install" -Verb RunAs -Wait; Remove-Item $Path\$Installer

2.  Next open the Chrome icon located on your desktop and log into the AWS account provided like you did initially for the workshop (using the Event Engine URL hash). From the AWS console, at the top of the screen, click **Services** and type & select **DataSync**

    -   Select **Get Started**

    -   In the Deploy agent section select **Amazon EC2** from the drop down, then
        click on the **Learn more**  icon in the same Deploy agent blue box

    -   Scroll down to the table that has a list of **AWS Regions and AMI Names**, and click on
        the **Launch Instance** link corresponding to the **us-east-1** row

        -   In the next page, select the box next to **m5.xlarge**

        -   Select **Next: Configure Instance Details**

        -   In the **Network** drop down select the VPC which has “**mod-xyz**”
            in its name

        -   In the **Subnet** drop down, select the one which has “**Public Subnet 0**”
            in its name

        -   Leave all other settings as default on the page

        -   Click **Next: Add Storage**

        -   Click **Next: Add Tags**

            -   Select **Add Tag**

        -   Enter the following values (case sensitive)

            -   Key = **Name**

            -   Value = **DataSync-agent**

        -   Click **Next: Configure Security Group**

            -   Click on the “**Select an existing security group**” check box

            -   Select the security groups with the names that start with
                of **mod-xyz**

        -   Click **Review and Launch**

        -   Click **Launch**

        -   From the select key pair drop down select **ee-default-keypair**, accept the check box and click **Launch
            Instances**  
            

    -   From the AWS console, click **Services** and type & select **EC2**

        -   From the left hand menu, select **Instances**

        -   In the right hand pane, select the box next to “**DataSync-agent**”

        -   From the bottom window pane, select the **Description** tab, and
            take note of the **private IP** address into the workshop notepad file you downloaded, as
             **DataSync Instance Private IP=**

            -   Ensure the “**Status Check**” column for this EC2 instance
                shows **“2/2 checks passed“** before proceeding to the next
                step.

        -   From the AWS console, at the top of the screen,
            click **Services** and type & select **DataSync**

            -   Select **Get Started**

            -   Select **Amazon EC2** from the **deploy agent** drop down, then enter the following values on the page

                -   **Service endpoint:** Select Public service endpoints in US
                    

                -   **Activation key:** Enter the Private IP address you noted
                    down in the previous step for **DataSync Instance Private IP**

                -   Select **Get Key**

                    -   You will get a notification when when your agent has activated successfully
                        agent has activated successfully

                -   Select **Create Agent** to continue

		

	-   When the create agent process is complete click on the **blue DataSync**
  	  link at the top left of the screen to continue with the next step of
  	  creating a transfer task


<br/><br/>


**Transfer data using DataSync** 
---------------------------------

1.  Following on from the previous AWS DataSync screen, click on the **Create task** from the top right hand side of the window

    -   Select **Create a new location** from the source locations options

    -   **Location type:** Server Message Block (SMB)

    -   **Agents:** select the agent you have just deployed

    -   **SMB Server:** enter your servers hostname that you took down 
        `e.g. EC2AMAZ-12ID25G`

    -   **Share name:**  enter **Tools**

    -   In the **Additional settings** select **SMB3** 
    -   In the User settings enter the user which has access to the source SMB share, enter the following values
	    -   User : **admin**
	    -   Password : The password for admin@example.com you used to log into your Windows EC2 instnace (from secret manager)
	    -   domain : **example**

    -   Click **Next** to continue
	    
2.  Select **Create a new location** from the Destination locations options

    -   **Location type:** Amazon FSx for Windows File Server

    -   **FSx file system:** Select the one you deployed (MAZ)

    -   **Share name:** enter **datasync**
    -   Click on the **Additional settings** and select from the **Security groups drop down menu** select the following groups : `FileSystemSecurityGroup`   and `ClientSecurityGroup`

    -   In the User settings enter the user which has access to the target SMB share, enter the following values and then click Next
	    -   User : **admin**
	    -   Password : The password for admin@example.com you used to log into your Windows EC2 instnace (from secret manager)
	    -   domain : **example**

    -   Click **Next** to continue

3.  Provide a task name (*i.e. local-smb-to-FSx-for-windows-file-server-transfer*)

    -   **Verify data:** Check integrity during transfer

    -   Leave the options as checked and scroll to the bottom of page to **Task logging** section and ensure the following perform the following

        -   **Log Level :** Select Log all transferred objects

        -   Click on the **Autogenerate** button on the screen to auto create a CloudWatch log group that will store the details of your files transferred


    -   Leave all other options & select **Next**

    -   Review the details and click **Create task**

4.  On the next screen wait until the **Task status** value is **Available**
    (refresh screen to get update)

5.  Click on the **Start** button

6.  Leave all options as they are (don’t override any) and click on **Start**

7.  At the top of the screen click on the **See execution details** button to
    view the progress of the transfer

8.  The task will go through a few phases, where it will perform some checks of the task setup, compare source and target details before starting the data transfer.


9.  When the **Execution status** show a status of **Success**, your data transfer has completed.

10.  Take note of the details shown such as **Duration** time taken for the data transfer, and also the performance details for this small transfer.
    

11. You can view the details of the files transferred by clicking on the **View logs in CloudWatch** button at the top right of the screen
12. In the next window pane look for an entry in the **Log streams** section, if there is no entry click the **refresh** circle icon, it can take a few minutes for the latest logs to be available.
13. When the log stream appears it will have a name similar to `task-0372cba912b113e4b-exec-xyz` click on it
14. Here you can view the details of the files transferred.



<br/><br/>

**Verify data transferred**
----------------------------------------------

Lets view the data copied across from the local SMB share to your target Amazon FSx for Windows File Server, file share.

1.  From your Windows EC2 instance open a windows file explorer to your **c:\Tools** folder and check out the size of the data and number of files stored
2.  Open a Windows command prompt (CMD) and enter the following command, and replace **your-fsx-DNS-name** with your value and press enter to map the target share

    `net use t: \\your-fsx-DNS-name\datasync`

3. Open a windows file explorer to your **T:\\** **drive** and check out the size of the data and number of files stored, does it match that of the source C:\Tools data?

<br/><br/>



**Summary**
-----------

In this module, you have obtained hands-on experience in deploying and configuring AWS
DataSync to simplify the transfer of SMB file share data to Amazon FSx for Windows File Server.

<br/><br/>

**END OF MODULE **
-------------------




