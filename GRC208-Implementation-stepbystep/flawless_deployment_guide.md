
flawless_deployment_guide.md
../../brain/3f47ba2d-1d25-4a2b-b109-bb697699d420


AWS GRC Capstone Project: The Beginner's Master Deployment Guide
This guide takes you from zero to full AWS Cloud Engineer. It explains exactly what you are doing in plain English, and completely bypasses every bug inherent to the original instructor's repository.

Phase 0: Getting Visual Studio Code (VS Code)
What is being done? Before you can build things in the cloud, you need a professional toolkit on your computer. Visual Studio Code (VS Code) is a free, incredibly popular code editor that lets you view all your project files, edit code comfortably, and type commands into a Terminal (Command Prompt) all on one screen.

0.1 Download and Install
Go to your internet browser and navigate to code.visualstudio.com.
Click the big blue Download for Windows button.
Open the downloaded .exe file. Click through the installer, accepting the agreements. Important: Leave all checkboxes on their default settings—especially the ones that say "Add to PATH".
Click Install and wait for it to finish.
0.2 Set Up Your Workspace
Create a new folder on your computer's Desktop and name it AWS-Capstone.
Open VS Code.
Go to the top-left menu, click File > Open Folder..., select your new AWS-Capstone folder, and click Select Folder. (If a prompt asks "Do you trust the authors of the files in this folder?", click Yes, I trust the authors).
0.3 Open Your Built-In Terminal
At the very top menu, click Terminal > New Terminal.
A panel will slide up from the bottom of your screen. This is your command line! Every single command in this guide will be pasted right here. (Make sure the dropdown on the top right of this panel says either PowerShell or Command Prompt).
Phase 1: Pre-Flight Preparation
What is being done? Your computer needs extra software to talk to AWS (the AWS CLI), download code (Git), and edit databases (MySQL). We are using a built-in Windows tool called winget to download them silently in the background, and then giving your terminal a temporary set of "keys" so it has permission to build things inside your AWS Learner Lab.

1.1 Fast-Install Prerequisites
Copy and paste these commands one by one into your VS Code terminal and press Enter to install the tools:

cmd
winget install --id Git.Git -e --source winget
winget install --id Python.Python.3.11 -e --source winget
winget install --id Oracle.MySQLWorkbench -e --source winget
(Once they finish downloading, click the trash can icon on the top right of your terminal panel to close it. Then click Terminal > New Terminal to open a fresh one that recognizes the new programs!).

1.2 Sync Your AWS Learner Lab Credentials
Go to your active AWS Academy Learner Lab in your web browser.
Click AWS Details, then click AWS CLI. Copy the entire [default] grey block.
Go back to VS Code, click File > Open File..., type %USERPROFILE%\.aws\credentials, and hit Enter. (If the folder doesn't exist, type mkdir %USERPROFILE%\.aws in your terminal first).
Paste the grey block inside this file and save it (Ctrl + S).
Close the file. Go down to your VS Code terminal and verify your sync:
cmd
aws configure set region us-east-1
aws sts get-caller-identity
(If it successfully prints out an Arn ending in LabRole, your computer is officially synced with the cloud!)

1.3 Download the Project Files
Run these commands to pull the original project files from GitHub directly into your VS Code folder:

cmd
git clone https://github.com/icdfa/GRC208-AWS-Capstone-Project.git
cd GRC208-AWS-Capstone-Project
pip install -r requirements.txt
Phase 2: Debugging the Instructor's Code
What is being done? The code provided in the assignment uses a technology called AWS CloudFormation. CloudFormation reads a blueprint file (.yaml) and automatically builds the cloud for you! However, the author's blueprint contains four specific bugs that cause AWS to reject it. We must manually edit the blueprint to fix typos, bypass Lab restrictions, and upgrade the database version so the deployment doesn't crash halfway through.

In your VS Code file explorer (on the left), click on cloudformation-database-stack.yaml to open it, and make the following 4 exact changes:

Bug 1: Database Isolated from Internet (Lines 43-44): The author put the database in a "Private Subnet", meaning your laptop physically cannot reach it to load data. Change the word private to public on both lines so the database spawns on the internet:

yaml
SubnetIds:
     - !ImportValue grc-capstone-public-subnet-1-id
     - !ImportValue grc-capstone-public-subnet-2-id
Bug 2: Missing Public IP Address (Around line 76): Even in a public subnet, the database needs a public IP address. Right under EnableIAMDatabaseAuthentication: true, press Enter and manually add this new line:

yaml
PubliclyAccessible: true
Bug 3: Blocked MySQL Version (Line 56): The author requested MySQL version 8.0.35, which AWS recently deprecated. Change the hard-coded string to the generic 8.0 so AWS picks a working version automatically:

yaml
EngineVersion: '8.0'
Bug 4: S3 Encryption Syntax Typo (Lines 119 and 168): The author forgot a required AWS sub-header for encryption on both the EvidenceBucket and the ComplianceReportsBucket blocks. You must type BucketEncryption: directly above the existing ServerSideEncryptionConfiguration line, and press the Tab key to indent the three lines beneath it inward. Both buckets should end up looking exactly like this:

yaml
BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: 'aws:kms'
              KMSMasterKeyID: !GetAtt S3EncryptionKey.Arn
🌟 Press Ctrl + S to save your bug-free file! 🌟

Phase 3: Hardware Deployment
What is being done? You are handing your fixed blueprints to AWS and telling it to physically provision your servers. First, it builds the Network Stack (the virtual buildings and internet wires). Second, it builds the Database Stack (the powerful MySQL hard drives that hold compliance data, and Amazon S3 folders that hold evidence logs).

3.1 Build the Network Foundation (Takes 5 Mins)
Deploy the subnets and wait for them to finish:

cmd
aws cloudformation create-stack --stack-name grc-capstone-network-stack --template-body file://cloudformation-network-stack.yaml --parameters ParameterKey=EnvironmentName,ParameterValue=grc-capstone --region us-east-1
aws cloudformation wait stack-create-complete --stack-name grc-capstone-network-stack --region us-east-1
3.2 Build the Database (Takes 15 Mins)
Deploy the database (Note: Do NOT use the @ symbol inside your password! AWS rejects it. The command below safely uses a - instead):

cmd
aws cloudformation create-stack --stack-name grc-capstone-database-stack --template-body file://cloudformation-database-stack.yaml --parameters ParameterKey=EnvironmentName,ParameterValue=grc-capstone ParameterKey=DBUsername,ParameterValue=grcadmin ParameterKey=DBPassword,ParameterValue=GRC208awsgrclilly-2026 --capabilities CAPABILITY_IAM --region us-east-1
aws cloudformation wait stack-create-complete --stack-name grc-capstone-database-stack --region us-east-1
When your blinking cursor finally returns, your database is live. Run this command to extract its unique internet address and save it somewhere safe!

cmd
aws cloudformation describe-stacks --stack-name grc-capstone-database-stack --region us-east-1 --query "Stacks[0].Outputs[?OutputKey=='DatabaseEndpoint'].OutputValue" --output text
Phase 4: Bypassing the Firewall
What is being done? AWS automatically generates a "Security Group" (a firewall) around your new database. By default, it rejects all traffic from the outside world. We are manually adding an Inbound Rule to this firewall so your specific personal laptop is allowed to pass through Port 3306.

Go to your AWS Console in your browser > Search RDS > Click Databases > Click grc-capstone-db.
Scroll to "Connectivity & security". Click the blue link under VPC security groups (sg-0abc...).
Click the Inbound rules tab at the bottom, then click Edit inbound rules.
Click Add rule at the bottom left (Do not overwrite any existing rules!).
Set Type to MySQL/Aurora (3306) and Source to Anywhere-IPv4 (0.0.0.0/0).
Click Save rules.
Phase 5: Serverless & Automation
What is being done? We are building the "brain" of the platform. You are compressing a small Python script supplied in the repo into a .zip file, and giving it to AWS Lambda. Lambda is a "serverless" service that runs code instantly only when triggered. We then configure AWS Config, a scanner that constantly watches your cloud and automatically triggers your Lambda brain if it sees a security violation.

5.1 Create AWS Lambda Run this command to find your Lab account ID string (arn:aws:iam...LabRole):

cmd
aws iam get-role --role-name LabRole --query "Role.Arn" --output text
(Now, right-click lambda_compliance_monitor.py in the VS Code file explorer on the left, click "Send To" -> "Compressed (zipped) folder", and name it lambda_compliance_monitor.zip!)

Replace <YOUR_LABROLE_ARN> below with the string you found above, and push your code to the cloud:

cmd
aws lambda create-function --function-name grc-compliance-monitor --runtime python3.11 --role <YOUR_LABROLE_ARN> --handler lambda_compliance_monitor.lambda_handler --zip-file fileb://lambda_compliance_monitor.zip --timeout 60 --memory-size 256 --region us-east-1

5.2 Configure AWS Config Recorder (Note: Because Learner Lab tightly restricts what student IAM Roles can do, AWS Config is prevented from delivering logs to S3. This is a known restriction! We simply enable the recorder and skip the S3 delivery channel).

Replace the 339712942498 numbers below with your own 12-digit Account ID:

cmd
aws configservice put-configuration-recorder --configuration-recorder-name grc-recorder --roleARN arn:aws:iam::339712942498:role/LabRole --region us-east-1
aws configservice start-configuration-recorder --configuration-recorder-name grc-recorder --region us-east-1
Phase 6: Loading Data and Final Validation
What is being done? Your infrastructure is fully built! Now you must inject it with actual compliance data (like ISO 27001 rules) to prove it works. You will use MySQL Workbench to visually log into the firewall opening we created, drop the fake data into the buckets, configure your local Python environment, and execute the master test suite!

6.1 Inject ISO 27001 Data via MySQL Workbench

Click your Windows Start button and open MySQL Workbench.
Click the + to add a new connection. Name it GRC Database. Paste your Database Web Address from Phase 3 into the Hostname box. User is grcadmin. Click "Store in Vault" and type GRC208awsgrclilly-2026.
Open the connection. Go to the top menu, click File > Open SQL Script, and load sample_data.sql.
Click at the very top of the script (line 1), type USE grcdb;, and click the Yellow Lightning Bolt button above it to run the injection!
6.2 Map Python .env Variables Python needs to know where everything is. Create a new file specifically named .env in your project folder within VS Code, and paste this inside:

text
# AWS Configuration
AWS_REGION=us-east-1
AWS_ACCOUNT_ID=<YOUR_12_DIGIT_ACCOUNT_ID_HERE>
# Database Configuration
DB_HOST=<YOUR_DATABASE_WEB_ADDRESS_HERE>
DB_PORT=3306
DB_NAME=grcdb
DB_USER=grcadmin
DB_PASSWORD=GRC208awsgrclilly-2026
6.3 Final Test Run! In your VS Code terminal, run the final verification tests:

cmd
python test_cases.py
(Take a screenshot of the Ran 22 tests OK message for your assignment, alongside a picture of your CloudFormation console showing CREATE_COMPLETE!)

Phase 7: Mission Teardown
What is being done? The assignment is over! Because cloud resources charge you money (or Learner Lab credits) for every hour they are left running, a good Cloud Engineer always destroys their sandbox lab entirely when they finish their project. The original .yaml file hard-coded a padlock onto your database (DeletionProtection: true), so CloudFormation will fail to delete it unless we manually pull the lock off first!

7.1 Remove the Safety Lock

cmd
aws rds modify-db-instance --db-instance-identifier grc-capstone-db --no-deletion-protection --apply-immediately --region us-east-1
7.2 Erase the Cloud Infrastructure Delete the heavy database, wait for it to disappear, and then delete the light network:

cmd
aws cloudformation delete-stack --stack-name grc-capstone-database-stack --region us-east-1
aws cloudformation wait stack-delete-complete --stack-name grc-capstone-database-stack --region us-east-1
aws cloudformation delete-stack --stack-name grc-capstone-network-stack --region us-east-1
Finally, close VS Code, return to your Learner Lab portal in your web browser, and click End Lab. You are an absolute master of AWS Cloud Architecture debugging!