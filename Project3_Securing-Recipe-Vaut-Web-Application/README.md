## AWS ARCHITECT NANODEGREE - PROJECT#3 (MYNOTES & REFERENCE FOR REVIEWER)
==========================================================================
### Review Comments Implementation
#### Comment-01: Good Job! However, your screenshot shows security group which allows anyone to communicate with your server over HTTP. Security Group should only allow traffic from port 5000 and subnet where load balancer resides.
* Solution: Done- Please see the screenshot attached. (E4T2_networksg.png updated)
![image](https://user-images.githubusercontent.com/13011167/85252211-ec320e00-b478-11ea-98e5-47d4b4cd4dec.png)
#### Comment-02: Great Job! Please also include Post-deployment compliance scanning in your diagram. (DevOpsPipeline.ppt Updated)
*  In a layered security approach, companies work to mitigate intrusion into their technology systems.Security in layers is the practice of applying multiple 
security measures, with each layer overlapping the previous and the next to create a net of security controls that work together to secure technology systems. I would categorize the security scanning into four categories:
   * Inline- Scanning: This first layer should include tools and scanners that take seconds (or maybe a few minutes) to run. Some common examples are code linters, 
     unit tests, static code analyzers like SonarQube, third-party dependency vulnerability checking like OWASP Dependency Checker, and possibly a subset of 
     integration tests.
   * Pre-deployment scanning: The second layer of DevSecOps includes tools that run inline with our deployment pipeline and take minutes to maybe an hour to 
     complete. This might include a more in-depth third-party vulnerability scan, Docker image scan, and malware scan.One key to this layer is that the scanners 
     and tools operate after the build artifacts are generated and before they are stored anywhere like Artifactory.
   * Post-deployment scanning: The tools in this layer include performance and integration testing and application scanners like OWASP Zap or AWS WAF. This layer 
     should run quickly, hopefully in an hour or less, to provide fast feedback to developers and limit the impact on our CD process.
   * Continuous scanning (Post Production): Most of the scanners are embedded in the CI/CD Pipeline. CS tools include tools like Nessus, Qualys, IBM App Scan, and 
     other infrastructure, application, and network scanning tools. CS surrounds it as an asynchronous, ongoing process and provides continual feedback to 
     developers. There are many vendors, such as Akamai or Cloudflare, which web clients point to as an entry point into your application. These are designated as 
     CDN providers, however they provide WAF capabilities as part of their offering. Cloud providers such as AWS also provide this type of service natively. AWS 
     WAF can be placed in front of public facing application services such as CloudFront and API Gateway transparently so that application endpoints do not need to 
     change.
#### Comment-03: Good Job with the tools. Please also share an example vulnerability for AWS Config. (E5T2.txt updated)
* Solution: 3. Scan an AWS environment for cloud configuration vulnerabilities
Solution: AWS Config (AWS Config evaluates whether your resource configurations comply with relevant rules and summarizes the compliance results)
### Example vulnerability
* EC2 or other resorces are compromised. Checks whether the incoming SSH traffic for the security groups is accessible. The rule is compliant when the IP addresses 
of the incoming SSH traffic in the security groups are restricted. 
* AWS Config collects configuration snapshots of many of the core infrastructure services that AWS provides. It provide insight into the resource is configured, 
when it changed, and what was changed. The configuration snapshot serves as an input to rules which evaluate whether the configuration is in compliance or out of 
compliance with the conditions specified in the rule. 
* AWS inspector is intended to specifically analyze and report on vulnerabilities on EC2 instances. AWS Inspector is designed to identify vulnerabilities at the OS 
level.Inspector can also provide a report on network reachability the public internet to instances and load balancers along with specific ports that are 
reachable.Once the inspector agent is installed on an instance, it can scan against:CIS benchmarks for linux and windows,AWS security best practices an Common 
vulnerabilities and exposures or CVE findings.

================================================================================

# Cloud Security - Secure the Recipe Vault Web Application
In this project, you will:
* Deploy and assess a simple web application environment’s security posture
* Test the security of the environment by simulating attack scenarios and exploiting cloud configuration vulnerabilities
* Implement monitoring to identify insecure configurations and malicious activity 
* Apply methods learned in the course to harden and secure the environment
* Design a DevSecOps pipeline
 
## Dependencies and Prerequisites
### Access to AWS account [Done]  
Students will need to use their personal AWS accounts.  Udacity will provide a $100 credit for any usage costs. If project instructions are followed we do not anticipate usage costs to exceed this amount.
 
### Installation of the AWS CLI and Local Setup of AWS API keys [Done]
Instructions and examples in this project will make use of the AWS CLI in order to automate and reduce time and complexity.
Refer to the below links to get the AWS CLI installed and configured in your local environment.
 
[Installing the CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
[Configuring the CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)
 
### Local setup of git and GitHub Repository 
You will need to clone or download [this GitHub repo](https://github.com/udacity/nd063-c3-design-for-security-project-starter) in order to work on and submit this project.

## Exercise 1 - Deploy Project Environment [Done]
**_Deliverables for Exercise 1:_**
* **E1T4.txt** - Text file identifying 2 poor security practices with justification. 
### Task 1:  Review Architecture Diagram
In this task, the objective is to familiarize yourself with the starting architecture diagram. An architecture diagram has been provided which reflects the resources that will be deployed in your AWS account.
 
The diagram file, title `AWS-WebServiceDiagram-v1-insecure.png`, can be found in the _starter_ directory in this repo.
 
![base environment](snapshots/AWS-WebServiceDiagram-v1-insecure.png)
 
#### Expected user flow:
- Clients will invoke a public-facing web service to pull free recipes.  
- The web service is hosted by an HTTP load balancer listening on port 80.
- The web service is forwarding requests to the web application instance which listens on port 5000.
- The web application instance will, in turn, use the public-facing AWS API to pull recipe files from the S3 bucket hosting free recipes. An IAM role and policy will provide the web app instance permissions required to access objects in the S3 bucket.
- Another S3 bucket is used as a vault to store secret recipes; there are privileged users who would need access to this bucket. The web application server does not need access to this bucket.
 
#### Attack flow:
- Scripts simulating an attack will be run from a separate instance which is in an un-trusted subnet.
- The scripts will attempt to break into the web application instance using the public IP and attempt to access data in the secret recipe S3 bucket.
#### Task-01:SOLUTION : Based on the architecture diagram, and the steps you have taken so far to upload data and access the application web service, identify at least 2 obvious poor practices as it relates to security.  Include justification.

```diff 
! Poor practice 1
+ Why Web Service Instance is in the public subnet(10.192.10.0/24)?Instead it should be in the private subnet(10.192.20.0/24).
+ Web Service should be hosted on private subnet and it will provide additional security. Also communication
+ channel between AppLoadBalancer and Web Service should be secure. I would prefer HTTPS commuication and + + will open the port 443 for ApploadBalancer. 
+ Security Group can be edited to accept traffic from + + ApploadBalancer only.

! Poor practice 2
+ Web Service for the purpose of retrieving the reciepies from S3 make use of Internet Gateway instead of using the role based methods to access the services within the AWS Infrastrcuture. 
+ The DefaultPrivateRoute1 and DefaultPrivateRoute2 tables have DestinationCidrBlock: 0.0.0.0/0
+ That means if a resource within the private subnets gets under malicious control, it could communicate with + any endpoint on the internet.
+ The InstanceRolePolicy-C3 for the IAM InstanceRole allows for S3 actions on any resource.
+ That means the IAM role could be used to access and change content in any S3 bucket in the AWS account.

```
### Task 2: Review CloudFormation Template [Done]
In this task, the objective is to familiarize yourself with the starter code and to get you up and running quickly. Spend a few minutes going through the .yml files in the _starter_ folder to get a feel for how parts of the code will map to the components in the architecture diagram. 
 
Additionally, we have provided a CloudFormation template which will deploy the following resources in AWS:
 
#### VPC Stack for the underlying network:
* A VPC with 2 public subnets, one private subnet, and internet gateways etc for internet access.
 
#### S3 bucket stack:
* 2 S3 buckets that will contain data objects for the application.
 
#### Application stack:
* An EC2 instance that will act as an external attacker from which we will test the ability of our environment to handle threats
* An EC2 instance that will be running a simple web service.
* Application LoadBalancer
* Security groups
* IAM role
#### Task-02:SOLUTION :
```diff 
! Reviewed the .yml files
+ Reviewed the cloudformation template to deploy the AWS resources as per the given architecture
```

### Task 3: Deployment of Initial Infrastructure [Done]
In this task, the objective is to deploy the CloudFormation stacks that will create the below environment.
 ![base environment](snapshots/AWS-WebServiceDiagram-v1-insecure.png)
 
We will utilize the AWS CLI in this guide, however you are welcome to use the AWS console to deploy the CloudFormation templates.
 
#### 1. From the root directory of the repository - execute the below command to deploy the templates.
##### Deploy the S3 buckets - c3-s3-goswami
```diff
aws cloudformation create-stack --region us-east-1 --stack-name c3-s3-goswami --template-body file://starter/c3-s3.yml
+ Command: aws cloudformation create-stack --region us-east-1 --stack-name c3-s3-goswami --template-body file://E:/'Amit Goswami'/Nanodegree-aws_architect/Project3_Securing-Recipe-Vaut-Web-Application/starter/c3-s3.yml
```
Expected example output:
```
{
    "StackId": "arn:aws:cloudformation:us-east-1:4363053XXXXXX:stack/c3-s3/70dfd370-2118-11ea-aea4-12d607a4fd1c"
}
```
##### Deploy the VPC and Subnets
```diff
aws cloudformation create-stack --region us-east-1 --stack-name c3-vpc --template-body file://starter/c3-vpc.yml
+ Command: aws cloudformation create-stack --region us-east-1 --stack-name c3-vpc-goswami --template-body file://E:/'Amit Goswami'/Nanodegree-aws_architect/Project3_Securing-Recipe-Vaut-Web-Application/starter/c3-vpc.yml
```
 
Expected example output:
```
{
    "StackId": "arn:aws:cloudformation:us-east-1:4363053XXXXXX:stack/c3-vpc/70dfd370-2118-11ea-aea4-12d607a4fd1c"
}
```
 
##### Deploy the Application Stack 
You will need to specify a pre-existing key-pair name.
```diff
aws cloudformation create-stack --region us-east-1 --stack-name c3-app --template-body file://starter/c3-app.yml --parameters ParameterKey=KeyPair,ParameterValue=<add your key pair name here> --capabilities CAPABILITY_IAM

+ Command: aws cloudformation create-stack --region us-east-1 --stack-name c3-app-goswami --template-body file://E:/'Amit Goswami'/Nanodegree-aws_architect/Project3_Securing-Recipe-Vaut-Web-Application/starter/c3-app.yml --parameters ParameterKey=KeyPair,ParameterValue=awsnanodegree --capabilities CAPABILITY_IAM
```
 
Expected example output:
```
{
    "StackId": "arn:aws:cloudformation:us-east-1:4363053XXXXXX:stack/c3-app/70dfd370-2118-11ea-aea4-12d607a4fd1c"
}
```
Expected example AWS Console status: https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks

![Expected AWS Console Status](snapshots/cloudformation_status.png)

#### 2. Once you see Status is CREATE_COMPLETE for all 3 stacks, obtain the required parameters needed for the project.
Obtain the name of the S3 bucket by navigating to the Outputs section of the stack:
 
![Outputs Section](snapshots/goswami_s3stack_output.png)
 
Note down the names of the two other buckets that have been created, one for free recipes and one for secret recipes.  You will need the bucket names to upload example recipe data to the buckets and to run the attack scripts.
 
- You will need the Application Load Balancer endpoint to test the web service - ApplicationURL(c1-web-service-alb-1429350102.us-east-1.elb.amazonaws.com)
- You will need the web application EC2 instance public IP address to simulate the attack - ApplicationInstanceIP(ec2-34-193-227-62.compute-1.amazonaws.com)
- You will need the public IP address of the attack instance from which to run the attack scripts - AttackInstanceIP(ec2-3-235-107-224.compute-1.amazonaws.com)
 
You can get these from the Outputs section of the **c3-app** stack.
 
![Outputs](snapshots/goswami_output.png)
 
#### 3.  Upload data to S3 buckets
Upload the free recipes to the free recipe S3 bucket from step 2. Do this by typing this command into the console (you will replace `<BucketNameRecipesFree>` with your bucket name):
 
Example:  
```
aws s3 cp free_recipe.txt s3://cand-c3-free-recipes-914415844393/ --region us-east-1
```
Upload the secret recipes to the secret recipe S3 bucket from step 2. Do this by typing this command into the console (you will replace `<BucketNameRecipesSecret>` with your bucket name):
 
Example:  
```
aws s3 cp secret_recipe.txt s3://cand-c3-secret-recipes-914415844393/ --region us-east-1
```
 
#### 4. Test the application
Invoke the web service using the application load balancer URL:
```diff
http://<ApplicationURL>/free_recipe
+ http://c1-web-service-alb-1429350102.us-east-1.elb.amazonaws.com/free_recipe

```
You should receive a recipe for banana bread.
The AMIs specified in the cloud formation template exist in the us-east-1 (N. Virginia) region. You will need to set this as your default region when deploying resources for this project. 
![goswami_test_application_url](snapshots/goswami_test_application_url.png)

### Task 4:  Identify Bad Practices
Based on the architecture diagram, and the steps you have taken so far to upload data and access the application web service, identify at least 2 obvious poor practices as it relates to security. List these 2 practices, and a justification for your choices, in the text file named E1T4.txt.
 
**Deliverables:**  [Done] - Enclosed the file in the Project03_Deliveriables for details.
- **E1T4.txt** - Text file identifying 2 poor security practices with justification. 
 
## Exercise 2: Enable Security Monitoring
 
**_Deliverables for Exercise 2:_**
- **E2T2_config.png** - Screenshot of AWS Config showing non-compliant rules.
- **E2T2_inspector.png** - Screenshot of AWS Inspector showing scan results.
- **E2T2.png_securityhub.png** - Screenshot of AWS Security Hub showing compliance standards for CIS foundations.
- **E2T2.txt** - Provide recommendations on how to remediate the vulnerabilities.
 
### Task 1: Enable Security Monitoring using AWS Native Tools
 
First, we will set up security monitoring to ensure that the AWS account and environment configuration is in compliance with the CIS standards for cloud security.
 
#### 1. Enable AWS Config (skip this step if you already have it enabled)  
 a. See below screenshot for the initial settings.   
 ![ConfigEnabled](starter/config_enable.png)  
 b. On the Rules page, click **Skip**.  
 c. On the Review page, click **Confirm**.
#### 2. Enable AWS Security Hub
 a. From the Security Hub landing page, click **Go To Security Hub**.  
b. On the next page, click **Enable Security Hub**
#### 3. Enable AWS Inspector scan
 a. From the Inspector service landing page, leave the defaults and click **Advanced**.  
 ![Inspector1](starter/inspector_setup_runonce.png)  
 b. Uncheck **All Instances** and **Install Agents**.  
 c. Choose Name for Key and ‘Web Services Instance - C3’ for value, click **Next**.  
 ![Inspector2](starter/inspector_setup_2.png)  
 d. Edit the rules packages as seen in the screenshot below.  
 ![Inspector3](starter/inspector_setup_3.png)  
 e. Uncheck **Assessment Schedule**.  
 f. Set a duration of 15 minutes.
 g. Click **Next** and **Create**.
#### 4. Enable AWS Guard Duty
a. After 1-2 hours, data will populate in these tools giving you a glimpse of security vulnerabilities in your environment.
#### E2-Task-01: [Done] 

### Task 2: Identify and Triage Vulnerabilities
Please submit screenshots of:
- AWS Config - showing non-compliant rules
- AWS Inspector - showing scan results
- AWS Security Hub - showing compliance standards for CIS foundations.
 
Name the files E2T2_config.png, E2T2_inspector.png, E2T2_securityhub.png respectively.
#### E2-Task-02:Snapshots : 
![E2T2_config](Project03_Deliverables/E2T2_config.png)
![E2T2_inspector](Project03_Deliverables/E2T2_inspector.png)
![E2T2_securityhub](Project03_Deliverables/E2T2_securityhub.png)
 
Research and analyze which of the vulnerabilities appear to be related to the code that was deployed for the environment in this project. Provide recommendations on how to remediate the vulnerabilities. Submit your findings in E2T2.txt
 
**Deliverables:** 
- **E2T2_config.png** - Screenshot of AWS Config showing non-compliant rules.
- **E2T2_inspector.png** - Screenshot of AWS Inspector showing scan results.
- **E2T2.png_securityhub.png** - Screenshot of AWS Security Hub showing compliance standards for CIS foundations.
- **E2T2.txt** - Provide recommendations on how to remediate the vulnerabilities.
```diff 
Research and analyze which of the vulnerabilities appear to be related to the code that was deployed for the environment in this project.
Security Hub Highlight some of the following top security vulnerabilities:
! HIGH
+ 2.1 Ensure CloudTrail is enabled in all regions
+ CloudTrail.1 CloudTrail should be enabled and configured with at least one multi-region trail
+ 4.2 Ensure no security groups allow ingress from 0.0.0.0/0 to port 3389
+ 4.1 Ensure no security groups allow ingress from 0.0.0.0/0 to port 22

! MEDIUM
+ IAM.7 Password policies for IAM users should have strong configurations
+ 2.9 Ensure VPC flow logging is enabled in all VPCs
+ IAM.5 MFA should be enabled for all IAM users that have console password
+ ELBv2.1 Application Load Balancer should be configured to redirect all HTTP requests to HTTPS
+ SSM.1 EC2 instances should be managed by AWS Systems Manager
+ S3.4 S3 buckets should have server-side encryption enabled

! LOW
+ API EnableSecurityHub was invoked using root credentials.

! Bonus - provide recommendations on how to remediate the vulnerabilities.
+ 1. Setting the appropiate IAM policy, do not use root user, create a child user and set the IAM policices for not allowing root user.
+ 2. Streamling the network communication between the entities by redirecting HTTP to HTTPS requests.
+ 3. Security Groups should be streamline and should not allow everything. 
+ 4. EC2 instance security groups should be alligned to only allow traffic from the specific entities as required.
+ 5. Enable VPC Flowlogs and s3 Bucket logging.
+ 6. TCP port 5000 is reachable from the internet on the EC2 instance. 
``` 
## Exercise 3 - Attack Simulation
Now you will run scripts that will simulate the following attack conditions:
Making an SSH connection to the application server using brute force password cracking.
Capturing secret recipe files from the s3 bucket using stolen API keys.
 
**_Deliverables for Exercise 3:_**
- **E3T1_guardduty.png** - Screenshot of Guard Duty findings specific to the Exercise 3, Task 1 attack.
- **E3T1.txt** - Answer to the questions at the end of Exercise 3, Task 1.
- **E3T2_s3breach.png** - Screenshot showing the resulting breach after the brute force attack.
- _Optional_ **Task 3** - Screenshots showing attack attempts and monitoring or logs from the WAF showing blocked attempts.
 
### Task 1: Brute force attack to exploit SSH ports facing the internet and an insecure configuration on the server

```
ssh -i <your private key file> ubuntu@<AttackInstanceIP>
```
The above instructions are for macOS X users.  For further guidance and other options to connet to the EC2 instance refer to [this guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
 
#### 1. Log in to the attack simulation server using your SSH key pair.
#### 2. Run the below commands to start a brute force attack against the application server.  You will need the application server hostname for this.
```diff
date
hydra -l ubuntu -P rockyou.txt ssh://<YourApplicationServerDnsNameHere>
+ hydra -l ubuntu -P rockyou.txt ssh://ec2-54-227-86-147.compute-1.amazonaws.com
```
You should see output similar to the following(My Snapshot after running above command):
![Brute Force](snapshots/goswami_brute_force.png)
 
Wait 10 - 15 minutes and check AWS Guard Duty.
 
#### 3. Answer the following questions:
1. What findings were detected related to the brute force attack?
#### i-0cbd0238825ee015e is performing SSH brute force attacks against 10.192.10.51. Brute force attacks are used to gain unauthorized access to your instance by guessing the SSH password.
2. Take a screenshot of the Guard Duty findings specific to the attack. Title this screenshot E3T1_guardduty.png.[Done]
3. Research the AWS Guard Duty documentation page and explain how GuardDuty may have detected this attack - i.e. what was its source of information?
```diff
+ Amazon GuardDuty : 
+ 1. Protect your AWS accounts and workloads with intelligent threat detection and continuous monitoring.It is a threat detection service that continuously monitors for 
+ malacious or unauthrorized behaviour to protect your AWS accounts and workloads. 
+ 2. With GuardDuty, we cna have an intelligent and cost-effective option for continuous threat detection in the AWS Cloud. The service uses machine learning, anomaly detection, 
+ and integrated threat intelligence to identify and prioritize potential threats. 
+ 3. GuardDuty analyzes tens of billions of events across multiple AWS data sources, such as AWS CloudTrail, Amazon VPC Flow Logs, and DNS logs and captured information aboit IP 
+ traffic going to and from EC2 network interfaces in VPC.
+ 4. GuardDuty alerts are actionable, easy to aggregate across multiple accounts, and straightforward to push into existing event management and workflow systems.
```
![guardduty](snapshots/goswami_guardduty.png)
 
Submit text answers in E3T1.txt.[Done]
 
**Deliverables:** [Done]
- **E3T1_guardduty.png** - Screenshot of Guard Duty findings specific to the Exercise 3, Task 1 attack.
- **E3T1.txt** - Answer to the questions at the end of Exercise 3, Task 1.
 
### Task 2: Accessing Secret Recipe Data File from S3
 
Imagine a scenario where API keys used by the application server to read data from S3 were discovered and stolen by the brute force attack.  This provides the attack instance the same API privileges as the application instance.  We can test this scenario by attempting to use the API to read data from the secrets S3 bucket.
 
#### 1. Run the following API calls to view and download files from the secret recipes S3 bucket.  You will need the name of the S3 bucket for this.
 
```diff
# view the files in the secret recipes bucket
aws s3 ls  s3://<BucketNameRecipesSecret>/ --region us-east-1
+ command: aws s3 ls  s3://cand-c3-secret-recipes-914415844393/ --region us-east-1

# download the files
aws s3 cp s3://<BucketNameRecipesSecret>/secret_recipe.txt  .  --region us-east-1
+ command: aws s3 cp s3://cand-c3-secret-recipes-914415844393/secret_recipe.txt  .  --region us-east-1

# view contents of the file
cat secret_recipe.txt
```
Take a screenshot showing the breach:
E3T2_s3breach.png
![E3T2_s3breach](Project03_Deliverables/E3T2_s3breach.png)

_Optional Stand Out Suggestion_ Task 3:
Choose one of the application vulnerability attacks outlined in the OWASP top 10 (e.g. SQL injection, cross-site scripting)
Attempt to invoke the application using the ALB URL with a corrupt or malicious URL payload.
Setup the AWS WAF in front of the ALB URL.
Repeat the malicious URL attempts
Observe the WAF blocking these requests.
Submit screenshots of your attempts and monitoring or logs from the WAF showing the blocked attempts.
* https://sucuri.net/guides/what-is-cross-site-scripting/
* https://www.google.com/about/appsecurity/learning/xss/
**Deliverables:** [Done]
- **E3T2_s3breach.png** - Screenshot showing the resulting breach after the brute force attack.
- _Optional_ **Task 3** - Screenshots showing attack attempts and monitoring or logs from the WAF showing blocked attempts.

## Exercise 4 - Implement Security Hardening
**_Deliverables for Exercise 4:_**
- **E4T1.txt** - Answer to the prompts in Exercise 4, Task 1.
- **E4T2_sshbruteforce.png** - Screenshot of terminal window showing the brute force attack and the remediation.
- **E4T2_networksg.png** - Screenshot of the security group change. 
- **E4T2_sshattempt.png** - Screenshot of your SSH attempt.
- **E4T2_s3iampolicy.png** - Screenshot of the updated IAM policy.
- **E4T2_s3copy.png** - Screenshot of the failed copy attempt.
- **E4T2_s3encryption.png** - screenshot of the S3 bucket policy
- **E4T3_securityhub.png** - Screenshot of Security Hub after reevaluating the number of findings.
- **E4T3_config.png** - Screenshot of Config after reevaluating the number of findings.
- **E4T3_inspector.png** - Screenshot of Inspector after reevaluating the number of findings.
- **E4T4.txt** - Answers from prompts in Exercise 4, Task 4.
- _Optional_ **c3-app_solution.yml** and **c3-s3_solution.yml** - Updated cloud formation templates which reflect changes made in E4 tasks related to AWS configuration changes.
- _Optional_ **E4T5.txt** - Additional hardening suggestions from Exercise 4, Task 5.

### Task 1 - Remediation plan
As a Cloud Architect, you have been asked to apply security best practices to the environment so that it can withstand attacks and be more secure.
1. Identify 2-3 changes that can be made to our environment to prevent an SSH brute force attack from the internet.
2. Neither instance should have had access to the secret recipes bucket; even in the instance that API credentials were compromised how could we have prevented access to sensitive data?

Submit answer in E4T1.txt
```diff
! Problem: # Identify 2-3 changes that can be made to our environment to prevent an ssh brute force attack from the internet.
+ Solution:
+ 1. SSH port should not be open to all 0.0.0.0/0. If we can restrict the SSH port so that outside entity should not be able to connect to the instance over SSH, it will help to 
+ make our EC2 instances secure.
+ 2. Security Group need to be changed and should only allow restricted access as per the least privilage policy.
+ 3. Disable SSH password login on the application server instance.

! Problem: # Neither instance should have had access to the secret recipes bucket, in the even that instance API credentials were compromised how could we have prevented access 
to sensitive data.
+ Solutions:
+ Need to follow the least privilage policy. Security groups need to be fine tuned to be able to access the S3 buckets with Free Recipes.
```
**Deliverables:**
- **E4T1.txt** - Answer to the prompts in Exercise 4, Task 1.

### Task 2 - Hardening
#### Remove SSH Vulnerability on the Application Instance
1. To disable SSH password login on the application server instance.

```diff
# open the file /etc/ssh/sshd_config
sudo vi /etc/ssh/sshd_config

# Find this line:
+ PasswordAuthentication yes

# change it to:
+ PasswordAuthentication no

# save and exit

#restart SSH server
sudo service ssh restart
```
2. Test that this made a difference.  Run the brute force attack again from Exercise 3, Task 1.  
3. Take a screenshot of the terminal window where you ran the attack highlighting the remediation and name it E4T2_sshbruteforce.png.

**Deliverables:**
- **E4T2_sshbruteforce.png** - Screenshot of terminal window showing the brute force attack and the remediation.
![E4T2_sshbruteforce](Project03_Deliverables/E4T2_sshbruteforce.png)

#### Apply Network Controls to Restrict Application Server Traffic
1. Update the security group which is assigned to the web application instance.  The requirement is that we only allow connections to port 5000 from the public subnet where the application load balancer resides.
2. Test that the change worked by attempting to make an SSH connection to the web application instance using its public URL.
3. Submit a screenshot of the security group change and your SSH attempt.

**Deliverables**:
- **E4T2_networksg.png** - Screenshot of the security group change. 
- **E4T2_sshattempt.png** - Screenshot of your SSH attempt.
![E4T2_networksg](Project03_Deliverables/E4T2_networksg.png)
![E4T2_sshattempt](Project03_Deliverables/E4T2_sshattempt.png)

#### Least Privilege Access to S3  
1. Update the IAM policy for the instance profile role used by the web application instance to only allow read access to the free recipes S3 bucket.
2. Test the change by using the attack instance to attempt to copy the secret recipes.
3. Submit a screenshot of the updated IAM policy and the attempt to copy the files. 

**Deliverables:**
- **E4T2_s3iampolicy.png** - Screenshot of the updated IAM policy.
- **E4T2_s3copy.png** - Screenshot of the failed copy attempt.
![E4T2_s3iampolicy](Project03_Deliverables/E4T2_s3iampolicy.png)
![E4T2_s3copy](Project03_Deliverables/E4T2_s3copy.png)

#### Apply Default Server-side Encryption to the S3 Bucket
This will cause the S3 service to encrypt any objects that are stored going forward by default.
Use the below guide to enable this on both S3 buckets.   
[Amazon S3 Default Encryption for S3 Buckets](https://docs.aws.amazon.com/AmazonS3/latest/dev/bucket-encryption.html)

Capture the screenshot of the secret recipes bucket showing that default encryption has been enabled.

**Deliverables**:
- **E4T2_s3encryption.png** - Screenshot of the S3 bucket policy.
![E4T2_s3encryption](Project03_Deliverables/E4T2_s3encryption.png)

### Task 3: Check Monitoring Tools to see if the Changes that were made have Reduced the Number of Findings
1. Go to AWS inspector and run the inspector scan that was run in Exercise 2.
2. After 20-30 mins - check Security Hub to see if the finding count reduced.
3. Check AWS Config rules to see if any of the rules are now in compliance.
4. Submit screenshots of Inspector, Security Hub, and AWS Config titled E4T3_inspector.png, E4T3_securityhub.png, and E4T3_config.png respectively.

**Deliverables**:
- **E4T3_securityhub.png** - Screenshot of Security Hub after reevaluating the number of findings.
- **E4T3_config.png** - Screenshot of Config after reevaluating the number of findings.
- **E4T3_inspector.png** - Screenshot of Inspector after reevaluating the number of findings.
![E4T3_securityhub](Project03_Deliverables/E4T3_securityhub.png)
![E4T3_config](Project03_Deliverables/E4T3_config.png)
![E4T3_inspector](Project03_Deliverables/E4T3_inspector.png)

### Task 4: Questions and Analysis
1. What additional architectural change can be made to reduce the internet-facing attack surface of the web application instance.
2. Assuming the IAM permissions for the S3 bucket are still insecure, would creating VPC private endpoints for S3 prevent the unauthorized access to the secrets bucket.
3. Will applying default encryption setting to the s3 buckets encrypt the data that already exists?
4. The changes you made above were done through the console or CLI; describe the outcome if the original cloud formation templates are applied to this environment?

Submit your answers in E4T4.txt.
**Deliverables**: [Done]
- **E4T4.txt** - Answers from prompts in Exercise 4, Task 4.
```diff
! Problem-01: # What additional architectural change can be made to reduce the internet facing attack surface of the web application instance.
+ Solution-01:
+ 1. We should block SSH logon to all EC2 instances and disable SSH logon altogether.
+ 2. We should all ports to the internet opened for all EC2 instances. Important internal access can only be allowed thorugh different Security groups as per the requirement of 
the project architecture.

! Problem-02: # Assuming the IAM permissions for the S3 bucket are still insecure, would creating VPC private endpoints for S3 prevent the unauthorized access to the secrets 
bucket.
+ Solution-02:
+ Creating VPC private endpoints would help to make the network communication faster and cheaper as data travel only within the AWS network and it does not require to make use 
+ of the internet gateway to route though the private IP. 
+ As per the current project architecture, secert recepies need to be accessed from the public network so short answer would be "NO".

! Problem-03: # Will applying default encryption setting to the s3 buckets encrypt the data that already exists?
+ Solution-03:
+ No, applying the default encryption to the S# bucket will not encrypt the alreadly present information in the S3 bucket.It can only encrypt the new objects added to the S3 
bucket.

! Problem-04: # What would happen if the original cloud formation templates are applied to this environment.
+ Solution-04: Cloud formation will over right all the changes and project architecture would be back to the original state. Only solution is to do the necessary changes in the 
"cloud formation" templates if we want to persist the code changes.

```
###  _Optional Standout Suggestion_ Task 5 - Additional Hardening
Make changes to the environment by updating the cloud formation template. You would do this by copying c3-app.yml and c3-s3.yml and putting your new code into c3-app_solution.yml and c3-s3_solution.yml.
Brainstorm and list additional hardening suggestions aside from those implemented that would protect the data in this environment. Submit your answers in E4T5.txt.

**Deliverables**:
- _Optional_ **c3-app_solution.yml** and **c3-s3_solution.yml** - updated cloud formation templates which reflect changes made in E4 tasks related to AWS configuration changes.
- _Optional_ **E4T5.txt** - Additional hardening suggestions from Exercise 4, Task 5.

## Exercise 5 - Designing a DevSecOps Pipeline
Take a look at a very common deployment pipeline diagrammed below:

![DevOpsPipeline](starter/DevOpsPipeline.png)

The high-level steps are as follows:
1. The user makes a change to the application code or OS configuration for a service.
2. Once the change is committed to source, a build is kicked off resulting in an AMI or a container image.
3. The infrastructure as code is updated with the new AMI or container image to use.
4. Changes to cloud configuration or infrastructure as code may have also been committed.
5. A deployment to the environment ensues applying the changes

**_Deliverables for Exercise 5:_**
- **DevSecOpsPipline.[ppt or png]** - Your updated pipeline. [Done]
- **E5T2.txt** - Answer from prompts in Exercise 5, Task 2. [Done]
- *Optional* **E5T3.png** - Screenshot of tool that has identified bad practices.
- *Optional* **E5T3.txt** - Answers from prompts in Exercise 5, Task 3.

### Task 1:  Design a DevSecOps pipeline
Update the starter DevOpsPipeline.ppt (or create your own diagram using a different tool)
At minimum you will include steps for:
- Infrastructure as code compliance scanning.
- AMI or container image scanning.
- Post-deployment compliance scanning.

Submit your design as a ppt or png image named DevSecOpsPipeline.[ppt or png].

**Deliverables**:
- **DevSecOpsPipline.[ppt or png]** - Your updated pipeline. [Done]
![image](https://user-images.githubusercontent.com/13011167/85231853-333be700-b418-11ea-8875-3ef10bf4b32e.png)

### Task 2 - Tools and Documentation
You will need to determine appropriate tools to incorporate into the pipeline to ensure that security vulnerabilities are found.
1. Identify tools that will allow you to do the following:
 a. Scan infrastructure as code templates.
 b. Scan AMI’s or containers for OS vulnerabilities.
 c. Scan an AWS environment for cloud configuration vulnerabilities.
2. For each tool - identify an example compliance violation or vulnerability which it might expose.

Submit your answers in E5T2.txt

**Deliverables**:
- **E5T2.txt** - Answer from prompts in Exercise 5, Task 2.

### _Optional Standout Suggestion_ Task 3 - Scanning Infrastructure Code

- Run an infrastructure as code scanning tool on the cloud formation templates provided in the starter.
- Take a screenshot of the tool that has correctly identified bad practices.
- If you had completed the remediations by updating the cloud formation templates, run the scanner and compare outputs showing that insecure configurations were fixed.

**Deliverables**:
- _Optional_ **E5T3.png** - Screenshot of tool that has identified bad practices.
- _Optional_ **E5T3.txt** - Answers from prompts in Exercise 5, Task 3.

## Exercise 6 - Clean up 

Once your project has been submitted and reviewed - to prevent undesired charges don’t forget to: 
- Disable Security Hub and Guard Duty.
- Delete recipe files uploaded to the S3 buckets.
- Delete your cloud formation stacks.

## _Optional Standout Suggestion_ Exercise 7 - Enjoying the Spoils of Your Good Security Work!

Bake one of the desserts from the recipe text files and submit a picture. :-)

### Important Links
#### Vulnerability Management: https://stackarmor.com/vulnerability-management-and-penetration-testing-on-aws-cloud/
