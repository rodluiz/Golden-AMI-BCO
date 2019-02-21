Skip to content
Search…
All gists
Back to GitHub
New gist
@rodluiz 
0 @rodluizrodluiz/Golden AMI Pipeline Lab.md
forked from danidoo/Golden AMI Pipeline Lab.md
Created 20 hours ago
 
<script src="https://gist.github.com/rodluiz/ac60e58ee797fa3db8a696325f6d9a43.js"></script>
  
 Code  Revisions 54
 Golden AMI Pipeline Lab.md
What is a Golden AMI
A golden AMI is an AMI that you standardize through configuration, consistent security patching, and hardening. It also contains agents you approve for logging, security, performance monitoring, etc. Customers have also expressed desire to establish repeatable processes to:

Distribute the golden AMI(s) to their business units.
Continuously assess the security posture of all active golden AMIs.
Decommission golden AMIs once obsolete.
About the golden AMI pipeline
The golden AMI pipeline enables creation, distribution, verification, launch-compliance, and decommissioning of the golden AMI out of the box. The following diagram highlights the high-level workflow. high-level AMI workflow

It is a standard DevOps best practice to establish golden AMIs (and the resulting running instances) as immutable objects and to manage any changes through a standard pipeline. Golden AMI pipeline follows the same best practice and enables the requirement of patching by allowing you to decommission an affected golden AMI version and creating a new one. Also, over time, a golden AMI version becomes obsolete. You can decommission the version by executing an automation set up by the pipeline.

Here is an architecture diagram of the golden AMI creation process: AMI Pipeline architecture diagram

Please follow the steps below to create a Golden Image pipeline and create, review and approve a Golden AMI.
Before Starting
The instructor will provide you an account, login instructions and will assign each one a region you'll be working on.
In this exercise we will use only the AMI Generation account.
For AMI Consumer account ID, you can use any 12 digit number like 111222333444.
Step A | Take note of your user ARN
Still in the AMI Generation account, open IAM -> Users. Roles (https://console.aws.amazon.com/iam/home#/users).
Open your User.
Take note of your User ARN:
Step B | Find on AWS Marketplace the latest version of the Deep Learning AMI (Amazon Linux) in your assigned region and take note of the AMI-ID
Choose EC2 in the Services menu (https://console.aws.amazon.com/ec2/v2/home)
Ensure that you are in your assigned region.
Choose Launch Instance.
In the search bar, type deep learning ami<enter>.
Take note of the Ami Id from the Deep Learning AMI (Amazon Linux) Version 21.2:
Step C | Setup the Golden AMI pipeline environment in the AMI Generation account with the following CloudFormation template:
https://github.com/aws-samples/aws-golden-ami-pipeline-sample/raw/master/Gold-AMi-Stack-CFT-CI.json

Open the following link, and download the JSON file to your computer: https://github.com/aws-samples/aws-golden-ami-pipeline-sample/raw/master/Gold-AMi-Stack-CFT-CI.json
Choose CloudFormation in the Services menu (https://console.aws.amazon.com/cloudformation/home?region=us-east-1)
Ensure that you are in your assigned region.
Choose Create Stack.
Choose Upload a template to Amazon S3.
Choose Browse and then choose the CloudFormation template you downloaded (Gold-AMi-Stack-CFT-CI.json)
Choose Next.
On the Specify Details page, specify a Stack name and the following parameters (values are case-sensitive)
Parameter	Value
Stack name	GoldenAMIPipelineStack
ApproverUserIAMARN	Approver ARN From Step B
EmailID	AMI Approver Email Address
productName	DL-1.0
productOSAndVersion	Linux-1.0
You can leave remaining parameters as it is. Choose Next
Choose Next
On the Review page, choose the check-box next to the following message: “I acknowledge that AWS CloudFormation might create IAM resources.”
Choose Create
After CloudFormation creates the stack, choose the check-box next to your stack and then choose Outputs tab in the panel on the bottom of the screen
Open your e-mail and confirm the two subscriptions for the Image Pipeline SNS Topics (Approver Notification and Continuous Assessment Results)
Step D | Create a golden AMI
Sign-in to the AWS Management Console using AMI Generation account credentials
Navigate to Systems Manager service (https://console.aws.amazon.com/systems-manager/home)
Ensure that you are in your assigned region.
Choose Execute Automation.
In the navigation panel, choose Automation under Actions drop-down.
Select Execute Automation
Click in the search bar, choose Owner and then Owned by me
Choose the GoldenAMIAutomationDoc document.
On Document version, choose Latest version at runtime
Choose Next
On the Input parameters table, specify:
Parameter	Value
sourceAMIid	DeepLearning AMI Id from Step C
You can leave remaining parameters as it is. Automation document launches instances in a private subnet, with a security group that has no inbound access for launching instances.
Next, choose Execute Automation. (~35 minutes for completion)
Step E | Verify Inspector findings and Approve the golden AMI
On the AMI Generation account, navigate to Systems Manager service.
Ensure that you are in your assigned region.
In the navigation panel, choose Parameter Store under Shared resources drop-down.
Filter parameters by clicking in the search bar, select Name and then begins-with and specify the path in the notification. (/GoldenAMI/Linux-1.0/DL-1.0/1/NumCVEs)
Choose the result.
Review the Value. The following value suggests that there were security findings found in the AMI.
To review findings, open the Inspector service console.
The dashboard will display Recent assessment runs. Choose the assessment run corresponding to your golden AMI.
You can either choose Download report to see details of the finding or you can review the information by choosing the number under the Findings column. Based on what you see, you can choose to Approve or Deny the AMI.
Navigate to Systems Manager service.
Ensure that you are in your assigned region.
In the navigation panel, choose Automation under Actions section.
You will see a list of automations, the automation that is ready for approval will have status as Waiting.
Choose the result corresponding to automation execution and then from Actions drop-down, choose Approve/Deny.
For this exercise, choose Approve.
Choose Submit.
Step F | Review the golden AMI metadata
After you approved the golden AMI, you will see a new private golden AMI registered under your AMIs in the Amazon EC2 console. You will also see metadata created for your golden AMI in Parameter Store. To view parameters created for your golden AMI:

On the AMI Generation account, navigate to Systems Manager service.
Ensure that you are in your assigned region.
In the navigation panel, choose Parameter Store under Shared resources drop-down.
Filter parameters by clicking in the search bar, select Name and then begins-with and specify the parameters you provided while creating a golden AMI:
/GoldenAMI/Linux-1.0/DL-1.0/1
Step G | Launch an EC2 instance with the Golden AMI
Navigate to EC2 service.
Ensure that you are in your assigned region.
Choose Launch Instance.
Choose My AMIs.
Choose Select next to the image created in Step E, DL-1.0-Linux-1.0-1.
Leave t2.micro instance type selected and choose Review and Launch.
Choose Launch.
Select Proceed without a key pair.
On the key pair selection page, choose the check-box next to the following message: "I acknowledge that I will not be able to connect to this instance unless I already know the password built into this AMI."
Choose Launch Instances
 @rodluiz
   
 
 
Leave a comment
Attach files by dragging & dropping, selecting them, or pasting from the clipboard.

 Styling with Markdown is supported
© 2019 GitHub, Inc.
Terms
Privacy
Security
Status
Help
Contact GitHub
Pricing
API
Training
Blog
About
Press h to open a hovercard with more details.
