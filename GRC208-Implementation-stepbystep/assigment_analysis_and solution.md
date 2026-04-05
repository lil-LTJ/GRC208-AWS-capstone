AWS GRC Capstone Project: Windows Learner Lab Solution Guide
1. Project Analysis
After reviewing the icdfa/GRC208-AWS-Capstone-Project repository, the goal of this capstone assignment is to deploy a comprehensive Governance, Risk, and Compliance (GRC) platform on AWS. The repository is presented as a fully developed, "ready-for-delivery" architecture and codebase (including CloudFormation templates, serverless functions, database schemas, and a React frontend).

The primary task for a student is not to write the code from scratch, but rather to successfully deploy, configure, and validate this enterprise-grade architecture within the heavily-managed AWS Academy Learner Lab environment.

2. Step-by-Step Deployment Solution
This solution is tailored for students using a Windows machine inside the AWS Academy Learner Lab.

Step 1: Secure Your AWS Environment (Learner Lab Specifics)
The AWS Academy Learner Lab provides a pre-configured, heavily managed environment. You must adapt standard instructions because of Lab restrictions:

Skip Billing Alerts: The lab has a pre-allocated limit (around $100). If you go over, it auto-stops. There is no personal billing risk.
Skip MFA Setup: You are using federated login via the academy portal, which does not support standard IAM MFA setups.
Skip Creating Custom IAM Users or Roles: You cannot create permanent IAM users or custom execution roles.
For CLI Access: Use the temporary credentials provided in your Learner Lab dashboard (click 'AWS Details' > 'AWS CLI'). These expire every 4 hours.
For IAM Roles: Whenever the repo assigns permissions (like for Lambda), use the pre-built LabRole provided by the Learner Lab.
Step 2: Prepare Your Local Environment
2.1 Install Prerequisites (Windows)

AWS CLI v2: Download and install the MSI from the AWS website. Verify via CMD: aws --version.
Python 3.8+: Download from python.org. Critical: Check the box "Add Python to PATH" during installation. Verify via CMD: python --version.
Git: Download from git-scm.com and install using default settings. Verify via CMD: git --version.
SQL Client: Install MySQL Workbench for visual database management.
2.2 Configure AWS CLI (Learner Lab Credentials)

Launch your AWS Learner Lab and click "Start Lab". Wait for the indicator to turn green.
Click AWS Details > AWS CLI and copy the [default] block containing aws_access_key_id, aws_secret_access_key, and aws_session_token.
On your computer, open C:\Users\YourName\.aws (create the .aws folder if missing). Create a text file named just credentials (no .txt extension!). Paste the copied block securely inside and save.
In Command Prompt, run:
cmd
aws configure set region us-east-1
aws configure set output json
Verify the connection: aws sts get-caller-identity.
2.3 Clone the Repository Open Command Prompt, navigate to your desired workspace (e.g., cd Desktop), and run:

cmd
git clone https://github.com/icdfa/GRC208-AWS-Capstone-Project.git
cd GRC208-AWS-Capstone-Project
2.4 Install Python Dependencies Inside the cloned folder, install the required libraries:

cmd
pip install -r requirements.txt
Step 3: Deploy the Infrastructure (CloudFormation)
3.1 Deploy the Network Stack

cmd
aws cloudformation create-stack --stack-name grc-capstone-network-stack --template-body file://cloudformation-network-stack.yaml --parameters ParameterKey=EnvironmentName,ParameterValue=grc-capstone --region us-east-1
Wait for it to finish (approx. 5 minutes):

cmd
aws cloudformation wait stack-create-complete --stack-name grc-capstone-network-stack --region us-east-1
3.2 Deploy the Database & Storage Stack Before running this, choose a secure password (must contain upper/lower case, numbers, and special characters). Replace YourSecurePassword123! below:

cmd
aws cloudformation create-stack --stack-name grc-capstone-database-stack --template-body file://cloudformation-database-stack.yaml --parameters ParameterKey=EnvironmentName,ParameterValue=grc-capstone ParameterKey=DBUsername,ParameterValue=grcadmin ParameterKey=DBPassword,ParameterValue=YourSecurePassword123! --capabilities CAPABILITY_IAM --region us-east-1
Wait for completion (approx. 10-15 minutes):

cmd
aws cloudformation wait stack-create-complete --stack-name grc-capstone-database-stack --region us-east-1
CRITICAL: Retrieve your new RDS Database Web Address and save it:

cmd
aws cloudformation describe-stacks --stack-name grc-capstone-database-stack --region us-east-1 --query "Stacks[0].Outputs[?OutputKey=='DatabaseEndpoint'].OutputValue" --output text
Step 4: Provision Serverless Automation (AWS Lambda)
Get Your LabRole ARN:
cmd
aws iam get-role --role-name LabRole --query "Role.Arn" --output text
Zip the Code: In Windows File Explorer, right-click lambda_compliance_monitor.py > "Send to" > "Compressed (zipped) folder". Name it lambda_compliance_monitor.zip.
Create the Function: Replace <YOUR_LABROLE_ARN> with the ARN from step 1:
cmd
aws lambda create-function --function-name grc-compliance-monitor --runtime python3.11 --role <YOUR_LABROLE_ARN> --handler lambda_compliance_monitor.lambda_handler --zip-file fileb://lambda_compliance_monitor.zip --timeout 60 --memory-size 256 --region us-east-1
Step 5: Configure AWS Config for Compliance Monitoring
Find your AWS Account ID: aws sts get-caller-identity --query "Account" --output text
Create a unique storage bucket (replace <YOUR_ACCOUNT_ID>):
cmd
aws s3 mb s3://grc-config-bucket-<YOUR_ACCOUNT_ID> --region us-east-1
Enable the Config Recorder (replace <YOUR_LABROLE_ARN>):
cmd
aws configservice put-configuration-recorder --configuration-recorder name=grc-recorder,roleARN=<YOUR_LABROLE_ARN> --region us-east-1
Turn the Scanner On:
cmd
aws configservice start-configuration-recorder --configuration-recorder-name grc-recorder --region us-east-1
Assign the Delivery Channel (replace <YOUR_ACCOUNT_ID>):
cmd
aws configservice put-delivery-channel --delivery-channel name=grc-channel,s3BucketName=grc-config-bucket-<YOUR_ACCOUNT_ID> --region us-east-1
Step 6: Initialize Database and Local Configuration
6.1 Seed the Database Connect to your new database endpoint and push the sample_data.sql script. Replace <YOUR_DATABASE_ENDPOINT>:

cmd
mysql -h <YOUR_DATABASE_ENDPOINT> -u grcadmin -p < sample_data.sql
(If mysql is not recognized, use MySQL Workbench GUI to run the SQL script instead).

6.2 Configure Local Project Environment Create a new file named exactly .env inside the project folder. Add the following, substituting your specific Account ID and Database Endpoint:

text
# AWS Configuration
AWS_REGION=us-east-1
AWS_ACCOUNT_ID=<REPLACE_ME_WITH_YOUR_ACCOUNT_ID>
# Database Configuration
DB_HOST=<REPLACE_ME_WITH_YOUR_DATABASE_ENDPOINT>
DB_PORT=3306
DB_NAME=grcdb
DB_USER=grcadmin
DB_PASSWORD=YourSecurePassword123!
6.3 Final Validation Run the automated test suite to verify the whole stack:

cmd
python test_cases.py
(Wait for the Ran 22 tests in x.xxxs OK success message).

Step 7: Post-Deployment Steps (Extra Credit / Professional Grade)
Monitoring Dashboard: aws cloudwatch put-dashboard --dashboard-name GRC-Platform --dashboard-body file://cloudwatch-dashboard.json
Automated Scheduled Checks:
cmd
aws events put-rule --name grc-compliance-check --schedule-expression "rate(1 hour)" --state ENABLED
aws events put-targets --rule grc-compliance-check --targets "Id"="1","Arn"="arn:aws:lambda:us-east-1:<YOUR_ACCOUNT_ID>:function:grc-compliance-monitor"
Audit Logging (CloudTrail):
cmd
aws s3 mb s3://grc-cloudtrail-logs-<YOUR_ACCOUNT_ID> --region us-east-1
aws cloudtrail create-trail --name grc-trail --s3-bucket-name grc-cloudtrail-logs-<YOUR_ACCOUNT_ID> --region us-east-1
aws cloudtrail start-logging --trail-name grc-trail --region us-east-1
Step 8: Finalizing & Teardown
8.1 Capture Screenshots (Proof of Submission) Embed the following screenshots with proper narrations into your final report:

Screenshot 1: The Passing Tests (Terminal output showing OK).
Narration: "This screenshot captures the successful execution of the test_cases.py script. The output confirms that all 22 automated integration tests passed in under a second. This is a critical validation step proving that the local Python environment successfully communicated with the deployed AWS infrastructure, verifying that the core backend logic is fully intact and operational."

Screenshot 2: CloudFormation Success (AWS Console showing green CREATE_COMPLETE).
Narration: "This screenshot displays the AWS CloudFormation dashboard, confirming that both the grc-capstone-database-stack and the grc-capstone-network-stack achieved a CREATE_COMPLETE status. This proves that the environment was deployed consistently, securely, and in strict alignment with the provided deployment templates."

Screenshot 3: AWS Config Dashboard (AWS Console showing "Recording is on").
Narration: "This screenshot illustrates the AWS Config dashboard with the configuration recorder actively running. This evidence demonstrates that the 'eyes' of the compliance system have been successfully enabled and are actively tracking the environment for any governance drift or security violations."

Screenshot 4: Amazon RDS Database (AWS Console showing grc-capstone-db as Available).
Narration: "This screenshot from the Amazon RDS dashboard shows the grc-capstone-db MySQL instance in an Available state. Its successful deployment ensures the application has a secure, persistent, and highly available data tier ready to retain audit evidence and compliance mappings."

8.2 Destroy Cloud Resources Run these to cleanly delete your lab components so you do not drain your budget:

cmd
aws cloudformation delete-stack --stack-name grc-capstone-database-stack --region us-east-1
aws cloudformation wait stack-delete-complete --stack-name grc-capstone-database-stack --region us-east-1
aws cloudformation delete-stack --stack-name grc-capstone-network-stack --region us-east-1
Finally, press the red End Lab button in your AWS Academy Portal to stop the session.