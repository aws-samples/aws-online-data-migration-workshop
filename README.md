GET HANDS-ON & LEARN BEST PRACTICES FOR ONLINE AWS DATA MIGRATIONS<br>
=======================================================================

Copyright Amazon Web Services, Inc. and its affiliates.All rights reserved. This sample code is made available under the MIT-0 license. See the LICENSE file.

Errors or corrections? Contact akbariw@amazon.com.

--------------------------------------------------------


OBJECTIVE OF WORKSHOP
--------------------------------

The prospect of moving data workloads to the cloud can be daunting, so can
trying to make sense of the array of tools, protocols, and mechanisms available
to move data into AWS.

In this workshop you will get hands-on experience in deploying, configuring and
transferring data at scale using some of the available AWS online & hybrid data tranfer services. 
In this lab we have a scenario where you will copy 10,000 small files to Amazon S3,
using different methods such as AWS S3 cp command script, AWS File Gateway, and also
AWS DataSync, where you will get an then access the data stored in Amazon S3 to better understand of the benefits of each service for different use cases. Lastly you will also get hands on experience with AWS Transfer for SFTP and using it as a transfer mechanism.

The image below illustrates at a high level the different 3 AWS Services that we will use use to get our data, into Amazon S3 in a simple, repeatable and efficient manner.
<img src="images/0-0.PNG">
<br/><br/>

**PREREQUISITES** 
--------------------------------

**AWS Account** – You will need an AWS account to log into to run this workshop, which has access to 
create AWS IAM roles, EC2 instances, AWS DataSync, AWS Storage Gateway, AWS Transfer for SFTP and CloudFormation stacks in the AWS regions you select.

**Browser** – It is recommended that you use the latest version of Chrome or
Firefox for this workshop.

**Remote Desktop Client** - You will need a RDP client to logon to the Windows
EC2 instance (Windows RDP)

**Key Pair** – You will need a valid EC2 Key Pair in the AWS region you choose
for your workshop (US-WEST-2 Oregon). Instructions are provided in this workshop
on generating and downloading an EC2 Key Pair.


<br/><br/>

**WORKSHOP MODULES**
--------------------

This workshop encompasses 4 modules

**Module 1** - Deploy resources

**Module 2** - AWS File Gateway

**Module 3** - AWS DataSync

**Module 4** - AWS Transfer for SFTP


<br>

To get started go to [module 1](/module1/README.md)
