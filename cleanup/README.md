CLEAN-UP WORKSHOP RESOURCES MODULE
==================================
Copyright Amazon Web Services, Inc. and its affiliates. All rights reserved.This sample code is made available under the MIT-0 license. See the LICENSE file.

Errors or corrections? Contact akbariw@amazon.com.

---------------------------------------------------------------------------------

**Note**: You will need to perform all these cleanup workshop instruction steps, from a browser
session from **YOUR LAPTOP/WORKSTATION** and not from the workshop's Windows Remote Desktop
Session which you will terminate as part of the clean up.


<br>

**DELETE THE AWS TRANSFER FOR SFTP INSTANCE YOU CREATED**
-----------------------------------------

1.  Navigate to the AWS console, at the top of the screen,
    click **Services** and type & select **AWS Transfer for SFTP**

2.  Select the **Server-ID** you created for the this workshop

3.  From the top menu select **Actions** and then **Delete**

4.  Type "delete" in the prompt field, and click on **Delete**

5.  Navigate to the AWS console, at the top of the screen,
    click **Services** and type & select **IAM**

6.  Click on **Roles** from the left hand menu

7.  On the right hand window, in the search bar, enter the name of the first IAM role you created in module 4 for the AWS Transfer SFTP role (i.e. **myAWSTransferUserRole**)

8. Select the box next to the name of your role from the search list, and click on **Delete role**

9. Cick on **Yes, delete**

10. Repeat steps 5 -9 for the second AWS Transfer for SFTP role you created in module 4 (**i.e. myAWSTransferLogRole**) then move on to the following step

11. Click on **Policies** from the left hand menu

12. On the right hand window, in the search bar, enter the name of the name of the IAM Policy you created in module 4 for the AWS Transfer SFTP user role (i.e. **target-s3bucket-rw-policy**)

13. Select the box next to the name of your policy from the search list, and click on **Policy actions** & **Delete**
149. Cick on **Delete**

<br/><br/>



**DELETE THE AMAZON KINESIS LAB ITEMS YOU CREATED**
-----------------------------------------

1.  Navigate to the AWS console, at the top of the screen from the search bar, 
    type and select **Cloud9**

2.  Select the environment you created (i.e. SaaS Producer), then click on **Delete**. Then confirm the delete operation

3.  Navigate to the AWS console, at the top of the screen from the search bar, 
    type and select **Amazon Kinesis**

4.  Expand the left hand menu and select **Delivery Streams**. Then select the delivery stream you created for this workshop (i.e. tenant_metrics_firehose), and click on **Delete**. Then confirm the delete operation

5.  Expand the left hand menu and select **Data Stream**. Then select the data stream you created for this workshop (i.e. tenant_metrics_stream).

6.  Click on **Actions** and **Delete**. Then confirm the delete operation

<br/><br/>



**DELETE THE TWO AMAZON S3 BUCKETS YOU CREATED**
-----------------------------------------

1.  Navigate to the AWS console, at the top of the screen,
    click **Services** and type & select **S3**

2.  Locate the bucket you created for **Source-S3-Bucket**

3.  Click on the check box next to the name

4.  Select **Delete** from the top options

5.  Follow the prompts

6.  Repeat step 2-5 for your **Target-S3-Bucket**

<br/><br/>


**DELETE AWS FILE GATEWAY NFS SHARES & GATEWAY**
--------------------------------------------

1.  Navigate to the AWS console, at the top of the screen,
    click **Services** and type & select **Storage Gateway**

2.  From the left hand menu select **File shares**

3.  Select the two NFS shares you created (check the box next to them)

4.  Click on **Actions** and  **Delete File share**

5.  Confirm deletion of resources and select **Delete**

6.  From the left hand menu select **Gateways**

7.  Select the File Gateway you deployed (**STG316-filegateway**)

8.  Click on **Actions** and  **Delete gateway**

9.  Confirm deletion of resources and select **Delete**

<br/><br/>


**DELETE AWS DATASYNC**
-----------------------

1.  Navigate to the AWS console, at the top of the screen,
    click **Services** and type & select **DataSync**

2.  From the left hand menu select **Tasks**

3.  Select the box next to the name of the task you created

4.  Click on **Action** from the drop down menu at the top of the screen and then **Delete*

5.  Confirm deletion of resources and select **Delete**

6.  From the left hand menu select **Locations**

7.  Select the two locations relating to your configuration

8.  Click on **Delete**

9.  Confirm deletion of resources and select **Delete**

10. From the left hand menu select **Agents**

11. Select the box next to the agent you deployed

12. Confirm deletion and click on **Delete**

<br/><br/>


**DELETE AMAZON EC2 RESOURCES DEPLOYED OUTSIDE OF CLOUDFORMATION**
-----------------------------------------------------------

1.  Navigate to the AWS console, at the top of the screen,
    click **Services** and type & select **EC2**

2.  From the left hand menu select **Instances**, then select the box next to the two EC2 instances you deployed, called **STG316-FileGateway** and **STG316-DataSync**

3.  Click on **Actions** then on **Instance state** and select **Terminate**

4.  Once you confirm these are the two EC2 instances you deployed for this workshop, Select **Yes,Terminate**

5.  From the left hand window pane select **Volumes**

6.  Click on the **refresh** icon on the top right hand corner until  the
    **150GB io1** volume for **STG316-filegateway** is showing its state as **available** in the **State** column.

7.  Verify that the **Attachment information** column for this volume is blank (i.e. not
    attached to any host)  
    

<img src="images/5-1.png">

8.  Select the box next to your **150GB io1** volume and click on **Actions** and  **Delete Volume**

9.  Confirm deletion of resources and select **Yes,Delete**

<br/><br/>


**DELETE CLOUDFORMATION STACK – STG316-RESOURCES**
--------------------------------------------------

1.  Navigate to the AWS console, at the top of the screen,
    click **Services** and type & select **CLOUDFORMATION**

2.  Select the **STG316-Resources** stack you deployed

<img src="images/5-2.png">

3.  Click on **Delete**

4.  Confirm deletion of resources and select **Delete stack** (this will take a few minutes)

5.  Next refresh your browser window to the cloudformation page or click on the refesh icon until your stack disappears from the list. (if it hasn't disappeared after 5 minute continue to the next step)

6.  Next click on the dropdown next to the search bar and change it from **Active** to **Deleted**

7.  Verify that your stack of **STG316-Resources** is showing a status of **DELETE_COMPLETE**

8.  Next click on the dropdown next to the search bar and change it from **Deleted** to **Active**

9.  From the same Cloudformation window select the box next to the **STG316-VPC** stack

10. Repeat steps 3-7

**CLEAN UP TASKS COMPLETED**
----------------------------
