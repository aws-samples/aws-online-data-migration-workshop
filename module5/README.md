Transferring SMB file data to Amazon FSx for Windows File Server using AWS DataSync
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
In this module you are going use AWS DataSync to migrate Windows based SMB file share data to a new Amazon FSx for Windows File Server Instance. You will firstly create a local Windows SMB file share on an Amazon EC2 compute instance, that will act as the source location for SMB file share data. You will then create an Amazon FSx for Windows File Server instance, and use it your target SMB file share location. Lastly, you will deploy and use AWS DataSync to migrate the source SMB file share data to the target FSx for Windows File Server, file share.

<br/><br/>

**Duration**
-------------------
It will take approximately 20 minutes to complete this section, and all work will be performed from the **Windows Server EC2 instance** you have used for this workshop.

<br/><br/>

**Deploy workshop data, and configure local Windows SMB Share**
-----------------------------

1. From the Windows EC2 instance that you are logged into for the workshop, left click on the Windows icon in the bottom left hand corner and type in **Notepad** and hit Enter. Use this notepad to write down values as requested for this workshop. 
2. Click on this using Internet Explorer (https://view.highspot.com/viewer/5f7a68e6b7b73918b28be6f9), and then click on the **download** button, when prompted select **Save** to download a powershell script, which will be used to deploy the your workshop items
3. Left click on the Windows icon in the bottom left corner and type in **Windows Powershell**, right click on the returned item and select **Run as administrator**, and  **YES** at the prompt. 
4. Let's go ahead a run the workshop script, in your powershell window, copy and paste the following commands one line at time and hit Enter: 
```
cd C:\users\Admin\Downloads\
Expand-Archive -Path deploy.zip
./deploy/deploy.ps1
```
5. At the security prompt, enter **R** and hit Enter to deploy the script which will install Google Chrome, create an SMB share, create a new Active Directory based user and also security group, and lastly create the sample 10,000 files that we will use for the transfer. 
6. Left click on the Windows icon in the bottom left hand corner and type **C:\\** and press Enter
6. Right click on the **Tools** folder and select **Properties**
7. Click on the **Security** tab, and then click on **Edit** **-->** **Add**
8. In the **Object name** field, copy and paste the `admin@example.com;file-users-group@example.com` and select **Check Names**, then select **OK**
9. On the next window pane, click on the **Admin** name, and from the permissions select **Full control**, and for **file-users-group** Select only **Read** and **List folder contents**, then select **OK** twice when prompted.
10. Left click on the Windows icon in the bottom left hand corner, type in **CMD**. In the command prompt type in **hostname** and press Enter. Copy down the value shown for your server's hostname in your the workshop notepad file, as you will need this later



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
8. Name the new folder **myshare**
9. Select OK
10. Complete the Create A Shared Folder Wizard, and at the **Shared folder permissions** screen, select **Custom permissions** and then click on **Custom** and enable **Everyone** with **Full Control**


<br/><br/>

**Deploy AWS DataSync agent**
-----------------------------

We are going to deploy the AWS DataSync agent within your AWS VPC as an EC2 instance. The AWS DataSync agent will then read directly from your Windows SMB share and transfer the data to your **Amazon FSx for Windows File Server, file share**


- Within your Windows EC2 instance, click on the Windows START icon in the bottom left hand corner and type **Powershell** and select **Windows Powershell**

Then copy and paste the below into the powershell window as a single line to download and install Google Chrome 

    $Path = $env:TEMP; $Installer = "chrome_installer.exe"; Invoke-WebRequest "http://dl.google.com/chrome/install/375.126/chrome_installer.exe" -OutFile $Path\$Installer; Start-Process -FilePath $Path\$Installer -Args "/silent /install" -Verb RunAs -Wait; Remove-Item $Path\$Installer
    
<br/>

1.  Next open the Chrome icon located on your desktop and log into the AWS account provided like you did initially for the workshop (using the Event Engine URL hash). From the AWS console, at the top of the screen, click **Services** and type & select **DataSync**

    -   Select **Get Started**

    -   In the Deploy agent section select **Amazon EC2** from the drop down, then
        click on the **Learn more**  icon in the same Deploy agent blue box

	-   Take note of the instructions for deploying the DataSync agent as an EC2 instance, which i have summarized for you with the follow steps. 
	-   Open a Windows Command Promt (CMD) and copy and paste the below command, where you replace the value of **$region** with your value (e.g. us-east-1) then run it, where it will return the value of the ami-id to use for your EC2 instance.

	`aws ssm get-parameter --name /aws/service/datasync/ami --region $region`

	- From the output of the above command copy the "ami" value shown for the item of **"Value"** into your workpad file as ami-id to use  in the next step (e.g. ami-0b4512915c43f10c9)
	
	**Note** : If you are unfamiliar with the region name syntax, you can use this command to list out the RegionName's (aws ec2 describe-regions --output table)

	- Next copy and paste the below command into your workshop notepad file and replace the **source-file-system-region** with your value (i.e. us-east-1) and also replace **ami-id** with the value you noted from above then, copy and paste that whole URL into a new tab in your Chrome Internet browser. 
	
	`https://console.aws.amazon.com/ec2/v2/home?region=source-file-system-region#LaunchInstanceWizard:ami=ami-id`
	
	`e.g. https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#LaunchInstanceWizard:ami=ami-0b4512915c43f10c9`

	This will now use this AMI image for our DataSync agent when we deploy it on our Amazon EC2 instance.
	


    -   Ok, now lets go ahead and configure the EC2 instance specifications before we deploy it. 

        -   In the next page, select the box next to **r5.xlarge**

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

    -   **Share name:**  enter **MyLocalShare**

    -   In the **Additional settings** select **SMB3** 
    -   In the User settings enter the user which has access to the source SMB share, enter the following values
	    -   User : **admin**
	    -   Password : The password for admin@example.com you used to log into your Windows EC2 instnace (from secret manager)
	    -   domain : **example**

    -   Click **Next** to continue
	    
2.  Select **Create a new location** from the Destination locations options

    -   **Location type:** Amazon FSx for Windows File Server

    -   **FSx file system:** Select the one you deployed (MAZ)

    -   **Share name:** enter **myshare**
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

    `net use t: \\your-fsx-DNS-name\myshare`

3. Open a windows file explorer to your **T:\\** **drive** and check out the size of the data and number of files stored, does it match that of the source C:\Tools data?

4. Lastly in the same file explorer window for **T:\\**, right click on a folder and select **Properties**, then select the **Security** tab. Take a look at the Active Directory based user and group permissions, did DataSync copy across the permissions you set on the source SMB share (at the start of this module) for the Active Directory user (admin) and group (file-users-group)?

<br/><br/>



**Summary**
-----------

In this module, you have obtained hands-on experience in deploying and configuring AWS
DataSync to simplify the transfer of SMB file share data to Amazon FSx for Windows File Server.

<br/><br/>

**END OF MODULE**
-------------------




