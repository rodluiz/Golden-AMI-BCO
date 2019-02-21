## What is a Golden AMI
A golden AMI is an AMI that you standardize through configuration, consistent security patching, and hardening. It also contains agents you approve for logging, security, performance monitoring, etc. Customers have also expressed desire to establish repeatable processes to:

- Distribute the golden AMI(s) to their business units.
- Continuously assess the security posture of all active golden AMIs.
- Decommission golden AMIs once obsolete.

## About the golden AMI pipeline
The golden AMI pipeline enables creation, distribution, verification, launch-compliance, and decommissioning of the golden AMI out of the box. The following diagram highlights the high-level workflow.
![high-level AMI workflow](https://d2908q01vomqb2.cloudfront.net/761f22b2c1593d0bb87e0b606f990ba4974706de/2018/05/16/GAP-1.png)

It is a standard DevOps best practice to establish golden AMIs (and the resulting running instances) as immutable objects and to manage any changes through a standard pipeline. Golden AMI pipeline follows the same best practice and enables the requirement of patching by allowing you to decommission an affected golden AMI version and creating a new one. Also, over time, a golden AMI version becomes obsolete. You can decommission the version by executing an automation set up by the pipeline.

Here is an architecture diagram of the golden AMI creation process:
![AMI Pipeline architecture diagram](https://d2908q01vomqb2.cloudfront.net/761f22b2c1593d0bb87e0b606f990ba4974706de/2018/05/16/GAP-process.png)

## Please follow the steps below to create a Golden Image pipeline and create, review and approve a Golden AMI.
Before Starting
===
1. The instructor will provide you an account, login instructions and will assign each one a region you'll be working on.
2. In this exercise we will use only the __AMI Generation__ account.
3. For __AMI Consumer__ account ID, you can use any 12 digit number like 111222333444.

Step A | Take note of your user ARN
===
1. Still in the __AMI Generation__ account, open __IAM__ -> __Users__.
   Roles (https://console.aws.amazon.com/iam/home#/users).
2. Open your __User__.
3. Take note of your __User ARN__:

Step B | Find on AWS Marketplace the latest version of the Deep Learning AMI (Amazon Linux) in your assigned region and take note of the AMI-ID
===
1. Choose __EC2__ in the Services menu (https://console.aws.amazon.com/ec2/v2/home)
2. Ensure that you are in your assigned region.
3. Choose __Launch Instance__.
4. In the search bar, type  __deep learning ami&lt;enter&gt;__.
5. Take note of the __Ami Id__ from the _Deep Learning AMI (Amazon Linux) Version 21.2_:

Step C | Setup the Golden AMI pipeline environment in the AMI Generation account with the following CloudFormation template:
===
  https://github.com/rodluiz/Golden-AMI-BCO/blob/master/Gold-AMi-Stack-CFT-CI.yaml
1. Open the following link, and  download the YAML file to your computer:
  https://github.com/rodluiz/Golden-AMI-BCO/blob/master/Gold-AMi-Stack-CFT-CI.yaml
2. Choose __CloudFormation__ in the Services menu (https://console.aws.amazon.com/cloudformation/home?region=us-east-1)
3. Ensure that you are in your assigned region.
4. Choose __Create Stack__.
5. Choose __Upload a template to Amazon S3__.
6. Choose __Browse__ and then choose the CloudFormation template you downloaded (Gold-AMi-Stack-CFT-CI.yaml)
7. Choose Next.
8. On the __Specify Details__ page, specify a Stack name and the following parameters (values are case-sensitive)

| Parameter | Value |
| --- | ----------------------------- |
| Stack name | GoldenAMIPipelineStack |
| ApproverUserIAMARN | _Approver ARN From Step B_ |
| EmailID | _AMI Approver Email Address_ |
| productName | DL-1.0 |
| productOSAndVersion | Linux-1.0 |

9. You can leave remaining parameters as it is. Choose __Next__
10. Choose __Next__
11. On the Review page, choose the __check-box__ next to the following message:
“I acknowledge that AWS CloudFormation might create IAM resources.”
12. Choose __Create__
13. After CloudFormation creates the stack, choose the check-box next to your stack and then choose __Outputs__ tab in the panel on the bottom of the screen
14. Open your e-mail and confirm the two subscriptions for the Image Pipeline SNS Topics (Approver Notification and Continuous Assessment Results)

Step D | Create a golden AMI
===
1. Sign-in to the AWS Management Console using __AMI Generation__ account credentials
2. Navigate to __Systems Manager__ service (https://console.aws.amazon.com/systems-manager/home)
3. Ensure that you are in your assigned region.
4. Choose __Execute Automation__.
5. In the navigation panel, choose __Automation__ under __Actions__ drop-down.
6. Select __Execute Automation__
7. Click in the search bar, choose __Owner__ and then __Owned by me__
8. Choose the __GoldenAMIAutomationDoc__ document.
9. On __Document version__, choose __Latest version at runtime__
10. Choose __Next__
11. On the __Input parameters__ table, specify:

| Parameter | Value |
| --- | --- |
| __sourceAMIid__ | _DeepLearning AMI Id from Step C_ |

12. You can leave remaining parameters as it is. Automation document launches instances in a private subnet, with a security group that has no inbound access for launching instances.
13. Next, choose __Execute Automation__. (~35 minutes for completion)

Step E | Verify Inspector findings and Approve the golden AMI
===
1. On the __AMI Generation__ account, navigate to __Systems Manager__ service.
2. Ensure that you are in your assigned region.
3. In the navigation panel, choose __Parameter Store__ under __Shared resources__ drop-down.
4. Filter parameters by clicking in the search bar, select __Name__ and then __begins-with__ and specify the path in the notification. (/GoldenAMI/Linux-1.0/DL-1.0/1/NumCVEs)
5. Choose the result.
6. Review the __Value__. The following value suggests that there were security findings found in the AMI.
7. To review findings, open the __Inspector__ service console.
8. The dashboard will display Recent assessment runs. Choose the assessment run corresponding to your golden AMI.
9. You can either choose Download report to see details of the finding or you can review the information by choosing the number under the Findings column. Based on what you see, you can choose to __Approve or Deny__ the AMI.
10. Navigate to __Systems Manager__ service.
11. Ensure that you are in your assigned region.
12. In the navigation panel, choose __Automation__ under Actions section.
13. You will see a list of automations, the automation that is ready for approval will have status as __Waiting__.
14. Choose the result corresponding to automation execution and then from __Actions__ drop-down, choose __Approve/Deny__.
15. For this exercise, choose __Approve__.
16. Choose __Submit__.

Step F | Review the golden AMI metadata
===
After you approved the golden AMI, you will see a new private golden AMI registered under your AMIs in the Amazon EC2 console. You will also see metadata created for your golden AMI in Parameter Store.
To view parameters created for your golden AMI:
1. On the __AMI Generation__ account, navigate to __Systems Manager__ service.
2. Ensure that you are in your assigned region.
3. In the navigation panel, choose __Parameter Store__ under __Shared resources__ drop-down.
4. Filter parameters by clicking in the search bar, select __Name__ and then __begins-with__ and specify the parameters you provided while creating a golden AMI:<br>
  `/GoldenAMI/Linux-1.0/DL-1.0/1`

Step G | Launch an EC2 instance with the Golden AMI
===
1. Navigate to __EC2__ service.
2. Ensure that you are in your assigned region.
3. Choose __Launch Instance__.
4. Choose __My AMIs__.
5. Choose __Select__ next to the image created in Step E, __DL-1.0-Linux-1.0-1__.
6. Leave __t2.micro__ instance type selected and choose __Review and Launch__.
7. Choose __Launch__.
8. Select __Proceed without a key pair__.
9. On the key pair selection page, choose the __check-box__ next to the following message:
  "I acknowledge that I will not be able to connect to this instance unless I already know the password built into this AMI."
10. Choose __Launch Instances__
