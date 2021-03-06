# Deploying MongoDB Replicaset Architecture in AWS Private VPC

This reference architecture provides a set of YAML templates for deploying the following AWS services :
- Amazon IAM
- Amazon Security Group
- Amazon EC2
- Amazon Route53

## Prerequisites Notes
The Cloudformation Security Group IP address is open by default (testing purpose). You should update the Security Group Access with your own IP Address to ensure your instances security.

Before you can deploy this process, you need the following:
 - Your AWS account must have one VPC available to be created in the selected region
 - Amazon EC2 key pair
 - Installed Domain in Route 53.
 - cloudformation-vpc (Assuming you already have installed VPC https://github.com/thinegan/cloudformation-vpc )

## We have test launch this CloudFormation stack in the following Region in our account:
 - US East (N. Virginia)

![infrastructure-overview](images/MongoDB_Replicaset_in_Private_VPC.png)

The repository consists of a set of nested templates that deploy the following:

 - A tiered [VPC](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html) with public and private subnets, spanning an AWS region.
 - A highly available ECS cluster deployed across two [Availability Zones](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) in an [Auto Scaling](https://aws.amazon.com/autoscaling/) group.
 - Two [NAT gateways](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-nat-gateway.html) to handle outbound traffic.
 - Two interconnecting microservices deployed as [ECS services](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html) (website-service and product-service). 
 - An [Application Load Balancer (ALB)](https://aws.amazon.com/elasticloadbalancing/applicationloadbalancer/) to the public subnets to handle inbound traffic.
 - ALB path-based routes for each ECS service to route the inbound traffic to the correct service.
 - Centralized container logging with [Amazon CloudWatch Logs](http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html).

## MongoDB Cloud Manager Setup

Create a New Project, click "New Project"
![CloudManager-Setup1](images/cm1_create_organization.png)

Select "Cloud Manager" and Click "Next"
![CloudManager-Setup2](images/cm2_create_project_1.png)

Enter Name of your project and click "Next"
![CloudManager-Setup3](images/cm2_create_project_2.png)

You project will be created.
![CloudManager-Setup4](images/cm2_create_project_3.png)

Goto Project "Deployment". Under "Crytera > Timeclonedbrep", select "Agents" and "Downloads & Settings".
Since, I'm using Debian Os, select Automation "Ubuntu (15.x, 16.x) - DEB"
![CloudManager-Setup5](images/cm3_create_agentid_1.png)

Use mmsGroupId and mmsApiKey to setup mms agent in your cloudformation script.
![CloudManager-Setup6](images/cm3_create_agentid_2.png)

A completed deployed mms automation agent running after completed cloudformation run. 
![CloudManager-Setup7](images/cm4_installed_automation_agent.png)

Goto Deployment > Security > Edit Setting.
Select "Authentication Mechanisms [X] Username/Password 
![CloudManager-Setup8](images/cm5_setup_authentication1.png)

Continue "Next" without enabling SSL. We will enable it on the process.
![CloudManager-Setup9](images/cm5_setup_authentication2.png)

Save and Initiate first Credential will be blank password.
Remember, you need re-run this credential process again to generate new password.
![CloudManager-Setup10](images/cm5_setup_authentication3.png)

Deploy you changes.
![CloudManager-Setup11](images/cm5_setup_authentication4.png)

Re-run the entire credential process again, only this time Agent mms-automation
user will generate a password. Don't Save and Deploy yet.
![CloudManager-Setup12](images/cm5_setup_authentication5.png)

Login to your Mongo Replica Master and create admin user first, based on the credential you got from Cloud Manager.
![CloudManager-Setup13](images/cm5_setup_authentication6.png)

Now, Save, Review and Deploy your changes,
![CloudManager-Setup14](images/cm5_setup_authentication7.png)

Next, Goto Deployment > Server.  
Install Monitoring Agent in Master Replica  
Install Monitoring and Backup Agent in Secondary Replica
![CloudManager-Setup15](images/cm6_setup_monitoring_backup1.png)

Confirm, Review and Deploy.
![CloudManager-Setup16](images/cm6_setup_monitoring_backup2.png)

Goto Deployment > Processes
Click "Manage Existing"
![CloudManager-Setup17](images/cm7_setup_Deployment1.png)

Add Master hostname and mongo port. Turn on "Enable Authentication".
![CloudManager-Setup18](images/cm7_setup_Deployment2.png)

Choose, Auth Mechanism "Username/Password".
Enter Username and Password. Select "Continue".
![CloudManager-Setup19](images/cm7_setup_Deployment3.png)

Continue but make sure you see all the processes in your deployment.
![CloudManager-Setup20](images/cm7_setup_Deployment4.png)

Check, "I understand that this require..." and click "Continue".
![CloudManager-Setup21](images/cm7_setup_Deployment5.png)

Check, "Yes, import users and roles from this deployment item".  
Click "Continue".
![CloudManager-Setup22](images/cm7_setup_Deployment6.png)

Proceed after "Automation Agent Successfully Verified".
![CloudManager-Setup23](images/cm7_setup_Deployment7.png)

Proceed after "Initialing Automation for your Deployment".
![CloudManager-Setup24](images/cm7_setup_Deployment8.png)

Save, Review and Deploy.
![CloudManager-Setup25](images/cm7_setup_Deployment9.png)

Replicaset Processes Display Completed!
![CloudManager-Setup26](images/cm7_setup_Deployment10.png)

Goto Deployment > Security > MongoDB User.  
Turn on "Enforce Consistent Set".  
Confirm "Enforce Consistent Set".
![CloudManager-Setup27](images/cm8_enforce_user_consistent1.png)

Save, Review and Deploy.
![CloudManager-Setup28](images/cm8_enforce_user_consistent2.png)

Now, Lets start the step to enable TLS/SSL setting.  
Please ensure you already have certs/pem install in your servers.  
Goto Deployment > Security > Authentication & TLS/SSL.  
Edit Setting and proceed to "Authentication Mechanisms" and Click "Next".
![CloudManager-Setup29](images/cm9_setup_ssl1.png)

Enable TLS/SSL option.  
Enter TLS/SSL CA File Path.  
Switch "Client Certificate Mode" to "Require".
![CloudManager-Setup30](images/cm9_setup_ssl2.png)

Enter PEM file for Automation, Backup and Monitoring Agent.  
Next Click "Save".
![CloudManager-Setup31](images/cm9_setup_ssl3.png)

Save, Review and Deploy.
![CloudManager-Setup32](images/cm9_setup_ssl4.png)

Proceed, Review and Deploy.
![CloudManager-Setup33](images/cm9_setup_ssl5.png)

Changes will shows as Enabled in TLS/SSL.
![CloudManager-Setup34](images/cm9_setup_ssl6.png)

Next, to Ensure the TLS/SSL support enabled in the Mongo replicaset,  
Goto Deployment > Processes. Select Replicaset Name and choose "Modify" setting.  
Update the Following:  
DB Directory Path Prefix = /data  
bindIp = 0.0.0.0  
sslMode = requireSSL  
sslPEMKeyFile = /etc/ssl/certs/mongodb.pem  
Then, click "Apply".
![CloudManager-Setup35](images/cm10_deploy_ssl1.png)

Now continue the previous step for the rest of the servers.
Mostly the update is just the following :  
sslMode = requireSSL  
sslPEMKeyFile = /etc/ssl/certs/mongodb.pem  
You will see the icon changes in your replicaset during this process.
![CloudManager-Setup36](images/cm10_deploy_ssl2.png)

Save, Review, Confirm and Deploy.
![CloudManager-Setup37](images/cm10_deploy_ssl3.png)

Proceed to Confirm and Deploy
![CloudManager-Setup38](images/cm10_deploy_ssl4.png)

Once Deploy is completed, you can double check the SSL/TLS changes by select a host and click the connect option to see example of connection command.
![CloudManager-Setup39](images/cm11_test_deployment1.png)

Click "Metric" to monitor all MongoDB Traffic/Usage.
![CloudManager-Setup40](images/cm12_monitor_cluster1.png)

Refer to "Data Explorer" for overall Data list.
![CloudManager-Setup41](images/cm13_test_data1.png)

Adding a New User. Click "Add New User".
![CloudManager-Setup42](images/cm14_add_user_role1.png)

Add the following.  
Identitier: test (dbname)  
username: user1  
Roles: dbOwner  
Password: xxxxxx  
Click "Add User".
![CloudManager-Setup43](images/cm14_add_user_role2.png)

Save, Review and Deploy.
![CloudManager-Setup44](images/cm14_add_user_role3.png)

Once changes take effects. You can double check your changes in your cli.
![CloudManager-Setup45](images/cm14_add_user_role4.png)

Troubleshoot Slow Query by Checking "Real Time" and check slowest operation.
![CloudManager-Setup46](images/cm15_monitor_slowquery1.png)

Also you can set log rotate from by your preference. 
![CloudManager-Setup47](images/cm16_setup_log1.png)

Finaly, you can remove the replicaset if you don't like and rebuild all over again.
![CloudManager-Setup48](images/cm17_remove_cloud_manager1.png)


#### Infrastructure-as-Code

A template can be used repeatedly to create identical copies of the same stack (or to use as a foundation to start a new stack).  Templates are simple YAML- or JSON-formatted text files that can be placed under your normal source control mechanisms, stored in private or public locations such as Amazon S3, and exchanged via email. With CloudFormation, you can see exactly which AWS resources make up a stack. You retain full control and have the ability to modify any of the AWS resources created as part of a stack. 

#### Self-documenting 

Fed up with outdated documentation on your infrastructure or environments? Still keep manual documentation of IP ranges, security group rules, etc.?

With CloudFormation, your template becomes your documentation. Want to see exactly what you have deployed? Just look at your template. If you keep it in source control, then you can also look back at exactly which changes were made and by whom.

#### Intelligent updating & rollback

CloudFormation not only handles the initial deployment of your infrastructure and environments, but it can also manage the whole lifecycle, including future updates. During updates, you have fine-grained control and visibility over how changes are applied, using functionality such as [change sets](https://aws.amazon.com/blogs/aws/new-change-sets-for-aws-cloudformation/), [rolling update policies](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatepolicy.html) and [stack policies](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/protect-stack-resources.html).

### Add a new item to this list

If you found yourself wishing this set of frequently asked questions had an answer for a particular problem, please [submit a pull request](https://help.github.com/articles/creating-a-pull-request-from-a-fork/). The chances are that others will also benefit from having the answer listed here.

## Contributing

Please [create a new GitHub issue](https://github.com/thinegan/cloudformation-mongodb/issues/new) for any feature requests, bugs, or documentation improvements. 

Where possible, please also [submit a pull request](https://help.github.com/articles/creating-a-pull-request-from-a-fork/) for the change. 

## Author

Thinegan Ratnam
 - [http://thinegan.com](http://thinegan.com/)

## Copyright and License

Copyright 2018 Thinegan Ratnam

Code released under the MIT License.