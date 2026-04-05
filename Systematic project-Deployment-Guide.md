# GRC Platform - Complete VS Code Deployment Guide

## Table of Contents
1. Prerequisites & Terminal Setup
2. Architecture Overview
3. Step-by-Step Deployment
4. Configuration
5. Testing and Validation
6. Post-Deployment Steps

## 1. Prerequisites & Terminal Setup

Since you are executing this project using **Visual Studio Code (VS Code)** on Windows, we need to install the required tools via the VS Code integrated terminal using `winget` (Windows Package Manager).

### 1.1 Fast-Install Required Software
Open your VS Code terminal (Terminal > New Terminal) and select **Command Prompt** or **PowerShell**, then paste these commands one by one to install the required software silently:

```powershell
# Install AWS CLI
winget install --id Amazon.AWSCLI -e --source winget
# Install Git
winget install --id Git.Git -e --source winget
# Install Python
winget install --id Python.Python.3.11 -e --source winget
# Install MySQL Workbench (for Database Management)
winget install --id Oracle.MySQLWorkbench -e --source winget
```
<span style="color:red;">
**Layman Explanation:** These commands act like an automatic app store for developers. Instead of manually downloading installers from Google, `winget` securely downloads and installs the fundamental tools (Terminal commands, Database viewers) directly onto your Windows machine.<br>
**Project Objective:** In order to manage cloud computers and test our compliance infrastructure locally, your laptop needs the same "languages" and tools that the AWS cloud uses.
</span>


> [!NOTE]
> **Important:** After these finish installing, you MUST click the trash can icon to close your terminal and open a brand new one (Terminal > New Terminal) so VS Code can recognize the newly installed `aws` and `python` commands.

### 1.2 Configure AWS Credentials
You are deploying to AWS Account ID: **630956767035**.

If you are using a standard IAM User:
```powershell
aws configure
# Paste your Access Key ID
# Paste your Secret Access Key
# Default region: us-east-1
# Default format: json
```
<span style="color:red;">
**Layman Explanation:** This acts as a logical "login screen" for your terminal. It stores your private security keys.<br>
**Project Objective:** Gives your local VS Code terminal the administrative permissions to remotely construct networks and databases inside your AWS account without needing to log in via a web browser.
</span>

Verify your computer is authenticated with AWS:
```powershell
aws sts get-caller-identity
```
<span style="color:red;">
**Layman Explanation:** This asks AWS, "Who am I logged in as right now?"<br>
**Project Objective:** It verifies that your terminal is successfully linked to your unique student AWS account ID, ensuring you don't accidentally deploy the Capstone project to a different environment.
</span>

---

### 1.3 Critical Modifications to CloudFormation Database Stack
Before deploying our infrastructure in Phase 1, you **must** make several technical modifications to the `cloudformation-database-stack.yaml` file. These changes are required to fix deployment bugs and strictly align with the AWS Free Tier limitations. 

Open `cloudformation-database-stack.yaml` in VS Code and make the following exact line-by-line changes:

#### 1. S3 Bucket Encryption Syntax Fixes (Lines 120 & 170)
* **What to change:** Find `BucketEncryption:` inside both the `EvidenceBucket` and `ComplianceReportsBucket` blocks. Make sure that the `ServerSideEncryptionConfiguration:` property is correctly indented **underneath** `BucketEncryption:`, pushing it "inward" so that it acts as a child of the encryption lock.
* **Why:** The original code had a schema syntax failure where the encryption configuration wasn't nested properly. By pushing it inwards, CloudFormation recognizes it as a valid security header, bypassing deployment crashes.

#### 2. Subnet Network Publicity (Lines 43-44)
* **What to change:** Change `- !ImportValue grc-capstone-private-subnet-1-id` to say `- !ImportValue grc-capstone-public-subnet-1-id`. (Do this for subnet 2 as well).
* **Why:** This forces our database into the public subnets instead of the private, unroutable network. It allows our local VS Code workbench to actually connect to the database over the internet.

#### 3. Database Engine Version Update (Line 56)
* **What to change:** Change `EngineVersion: '8.0.35'` to just say `EngineVersion: '8.0'`.
* **Why:** AWS recently deprecated the highly specific `8.0.35` version, meaning asking for it will cause an instant failure. Switching it to `8.0` tells AWS to just automatically pull their most recent stable version.

#### 4. Free-Tier Compliance Restrictions (Lines 68 & 71)
* **What to change:** Change `BackupRetentionPeriod: 7` to `BackupRetentionPeriod: 1`. Then change `MultiAZ: true` to `MultiAZ: false`.
* **Why:** These are massive cost-savers. `MultiAZ: true` creates a second, identical backup database in another data center, which instantly falls outside the AWS Free Tier and triggers hourly billing. Reducing backups from 7 days to 1 day drastically limits storage footprint costs.

#### 5. Enable Public Accessibility (Line 77)
* **What to change:** Add the line `PubliclyAccessible: true` directly under the `EnableIAMDatabaseAuthentication: true` line.
* **Why:** Even though we put the database in a public subnet earlier, AWS databases default to rejecting external internet traffic as a security measure. Flipping this flag to `true` is the final step required so we can securely seed the database from our local laptop.

---

## 2. Architecture Overview
**Network Layer:** VPC with public subnets, Internet Gateway.
**Data Layer:** Amazon RDS MySQL 8.0, Amazon S3.
**Application Layer:** AWS Lambda.
**Security Layer:** AWS Config, AWS CloudTrail.

---

## 3. Step-by-Step Deployment

Navigate into the project repository if you haven't already:
```powershell
cd d:\GRC208-AWS-capstone\GRC208-AWS-capstone-2
```

### Phase 1: Network Infrastructure

Deploy the network stack which builds the VPC and Subnets:

```powershell
aws cloudformation create-stack `
  --stack-name grc-capstone-network-stack `
  --template-body file://cloudformation-network-stack.yaml `
  --parameters ParameterKey=EnvironmentName,ParameterValue=grc-capstone `
  --region us-east-1

# Wait for stack to complete (takes ~5 mins)
aws cloudformation wait stack-create-complete `
  --stack-name grc-capstone-network-stack `
  --region us-east-1
```
<span style="color:red;">
**Layman Explanation:** AWS CloudFormation acts like a 3D-Printer for cloud resources. The first command gives AWS a blueprint (`.yaml` file) to instantly build your virtual networking cables, routers, and firewalls (VPC & Subnets). The second command is a "loading screen" that forces the terminal to freeze until construction is 100% complete.<br>
**Project Objective:** Security and Compliance start at the network tier. This isolates our backend data away from the public internet, making it compliant with baseline enterprise security standards.
</span>


### Phase 2: Database Infrastructure

Deploy the database stack. We will set the default password here.

```powershell
aws cloudformation create-stack `
  --stack-name grc-capstone-database-stack `
  --template-body file://cloudformation-database-stack.yaml `
  --parameters `
    ParameterKey=EnvironmentName,ParameterValue=grc-capstone `
    ParameterKey=DBUsername,ParameterValue=grcadmin `
    ParameterKey=DBPassword,ParameterValue='GRC208awsgrclilly-2026' `
  --capabilities CAPABILITY_IAM `
  --region us-east-1

# Wait for stack completion (takes ~15 mins)
aws cloudformation wait stack-create-complete `
  --stack-name grc-capstone-database-stack `
  --region us-east-1
```
<span style="color:red;">
**Layman Explanation:** This submits a second blueprint to the AWS 3D-Printer. This time, it manufactures a full MySQL Database Server, injects your custom master password, and plugs it into the secure network we just built.<br>
**Project Objective:** To demonstrate data governance, we need a central repository to hold our compliance risk audits and system controls. This RDS database acts as our centralized source of truth.
</span>


Extract the database connection endpoint for your records:
```powershell
$DB_ENDPOINT = aws cloudformation describe-stacks `
  --stack-name grc-capstone-database-stack `
  --region us-east-1 `
  --query "Stacks[0].Outputs[?OutputKey=='DatabaseEndpoint'].OutputValue" `
  --output text
  
echo $DB_ENDPOINT
```
<span style="color:red;">
**Layman Explanation:** This queries the finished database and says, "What is your public URL domain name?" and prints it to the screen.<br>
**Project Objective:** You physically cannot upload data into a database if you don't know its address. We capture this endpoint so we can target it with our database injection tools in Phase 4.
</span>


### Phase 3: Lambda Functions

For the Lambda component, we create a role using a JSON trust policy file (PowerShell breaks inline JSON), zip our python code, and upload it.

> **Note:** PowerShell breaks inline JSON, so we write trust policies to `.json` files first and reference them with `file://`.

First, create the Lambda trust policy file:
```powershell
# Step 1: Create the trust policy JSON file
@'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
'@ | Out-File -Encoding ascii lambda-trust-policy.json
```
<span style="color:red;">
**Layman Explanation:** This creates a text file containing "IAM Trust Policy" rules. It explicitly tells AWS, "I promise it's okay for the Lambda service to borrow these credentials."<br>
**Project Objective:** AWS follows a "Zero Trust" model. Without this explicit trust document, our Lambda function physically cannot be executed by AWS services, halting our automated monitoring.
</span>

Now create the role and attach policies:
```powershell
# Step 2: Create IAM role for Lambda using the JSON file
aws iam create-role --role-name grc-lambda-role --assume-role-policy-document file://lambda-trust-policy.json

# Step 3: Attach all required policies to the role
aws iam attach-role-policy --role-name grc-lambda-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam attach-role-policy --role-name grc-lambda-role --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam attach-role-policy --role-name grc-lambda-role --policy-arn arn:aws:iam::aws:policy/AWSConfigUserAccess

# Step 4: Retrieve the role ARN (copy this â€” you'll need it next)
aws iam get-role --role-name grc-lambda-role --query "Role.Arn" --output text
```
<span style="color:red;">
**Layman Explanation:** This builds a digital "ID badge" for our code (the IAM Role) and glues 4 specific permission stickers onto it (Write logs, Write to S3, Read Configs, Execute). It then prints the badge's serial number.<br>
**Project Objective:** Our GRC Lambda script needs permission to gather evidence and store the audit reports. This exact blend of policies fulfills the "Principle of Least Privilege" required by compliance standards.
</span>

You should see: `arn:aws:iam::630956767035:role/grc-lambda-role`

```powershell
# Step 4: Wait for IAM to propagate
Start-Sleep -Seconds 15

# Step 5: Zip the function using PowerShell
Compress-Archive -Path lambda_compliance_monitor.py -DestinationPath lambda_compliance_monitor.zip -Force

# Step 6: Create the Lambda function
aws lambda create-function `
  --function-name grc-compliance-monitor `
  --runtime python3.11 `
  --role arn:aws:iam::630956767035:role/grc-lambda-role `
  --handler lambda_compliance_monitor.lambda_handler `
  --zip-file fileb://lambda_compliance_monitor.zip `
  --timeout 60 `
  --memory-size 256 `
  --region us-east-1
```
<span style="color:red;">
**Layman Explanation:** This zips up our Python script and mails it to the AWS Lambda servers to be hosted. We attach the ID badge we just made to give it permissions.<br>
**Project Objective:** This script is the "engine" of our automated compliance mechanism. By hosting it serverlessly via Lambda, we eliminate OS patching vulnerabilities, a huge benefit for security scoring.
</span>

### Phase 4: AWS Config Setup

First, create the Config trust policy file:
```powershell
# Create the trust policy JSON file for Config
@'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "config.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
'@ | Out-File -Encoding ascii config-trust-policy.json
```
<span style="color:red;">
**Layman Explanation:** Very similar to the Lambda trust file, but this says AWS Config service is allowed to act on your behalf.
</span>

```powershell
# Create IAM role for Config using the JSON file
aws iam create-role --role-name grc-config-role --assume-role-policy-document file://config-trust-policy.json

# Attach AWS managed policy
aws iam attach-role-policy `
    --role-name grc-config-role `
    --policy-arn "arn:aws:iam::aws:policy/service-role/AWS_ConfigRole"

Start-Sleep -Seconds 10

# Create S3 Bucket (Name is globally unique! We attach your account ID)
aws s3 mb s3://grc-config-bucket-630956767035 --region us-east-1

# Create the bucket policy file (grants AWS Config permission to write to the bucket)
@'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSConfigBucketPermissionsCheck",
      "Effect": "Allow",
      "Principal": { "Service": "config.amazonaws.com" },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::grc-config-bucket-630956767035"
    },
    {
      "Sid": "AWSConfigBucketDelivery",
      "Effect": "Allow",
      "Principal": { "Service": "config.amazonaws.com" },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::grc-config-bucket-630956767035/*",
      "Condition": {
        "StringEquals": { "s3:x-amz-acl": "bucket-owner-full-control" }
      }
    }
  ]
}
'@ | Out-File -Encoding ascii config-bucket-policy.json

# Apply the bucket policy so AWS Config can deliver logs
aws s3api put-bucket-policy `
  --bucket grc-config-bucket-630956767035 `
  --policy file://config-bucket-policy.json

# Enable AWS Config
aws configservice put-configuration-recorder `
  --configuration-recorder name=grc-recorder,roleARN=arn:aws:iam::630956767035:role/grc-config-role `
  --region us-east-1

# Create delivery channel
aws configservice put-delivery-channel `
  --delivery-channel name=grc-channel,s3BucketName=grc-config-bucket-630956767035 `
  --region us-east-1

# Start recording
aws configservice start-configuration-recorder `
  --configuration-recorder-name grc-recorder `
  --region us-east-1
```
<span style="color:red;">
**Layman Explanation:** This massive block creates a secure locker (an S3 storage bucket), sets up a security guard (AWS Config) with an ID badge, and tells the guard to start recording all infrastructure changes and dumping the "security camera footage" into the locker.<br>
**Project Objective:** Audit trails are a cornerstone of Governance, Risk, and Compliance. Setting up the AWS Config deliverable pipeline ensures every configuration drift is immutably logged for compliance auditors.
</span>

---

## 4. Database Optimization & Data Loading

### Option A: Using MySQL Workbench (VS Code / Local Machine)

Now that infrastructure is live, open **MySQL Workbench** (which you installed in step 1.1) to inject the data:

1. Click the **+** (Add connection).
2. Hostname: Paste the `$DB_ENDPOINT` URL you captured.
3. Username: `grcadmin`
4. Password: Click **Store in Vault** and type `GRC208awsgrclilly-2026`
5. Click **OK**, then double click the connection to open it.
6. Go to **File > Open SQL Script** and pick `sample_data.sql` inside the repo folder.
7. Type `USE grcdb;` on Line 1 of the SQL file, then click the **Yellow Lightning Bolt** button to inject the data.

---

### Option B: Using AWS CloudShell (Browser-Based â€” No Local Tools Needed)

If MySQL Workbench is not available or you hit a connection timeout, use **AWS CloudShell** â€” it runs inside AWS so there are no firewall or security group issues.

> [!WARNING]
> CloudShell uses **bash**, NOT PowerShell. Do NOT use backtick (`` ` ``) line continuations here â€” use backslash (`\`) instead. Mixing them up causes `--region: command not found` errors.

**Step 1:** In your AWS Console, click the **CloudShell icon `>_`** at the top right of the page.

**Step 2:** Clone the project repo into CloudShell so you have the `sample_data.sql` file:
```bash
git clone https://github.com/icdfa/GRC208-AWS-Capstone-Project.git
cd GRC208-AWS-Capstone-Project
```

**Step 3:** Install the MySQL client inside CloudShell:
```bash
sudo yum install -y mysql
```

**Step 4:** Get your database endpoint â€” use backslash `\` continuations (bash syntax):
```bash
DB_ENDPOINT=$(aws cloudformation describe-stacks \
  --stack-name grc-capstone-database-stack \
  --region us-east-1 \
  --query "Stacks[0].Outputs[?OutputKey=='DatabaseEndpoint'].OutputValue" \
  --output text)

echo $DB_ENDPOINT
```

**Step 5:** Fix the Security Group BEFORE connecting (otherwise you will get `ERROR 2002`):

1. Go to **AWS Console â†’ RDS â†’ Databases â†’ grc-capstone-db**
2. Click the **Connectivity & Security** tab
3. Click the blue **VPC security group** link (e.g. `sg-0abc...`)
4. Click **Inbound rules** tab â†’ **Edit inbound rules**
5. Click **Add rule**:
   - **Type:** `MySQL/Aurora`
   - **Port:** `3306`
   - **Source:** `Anywhere-IPv4 (0.0.0.0/0)`
6. Click **Save rules**

**Step 6:** Connect and load the sample data:
```bash
mysql -h $DB_ENDPOINT -u grcadmin -pGRC208awsgrclilly-2026 < sample_data.sql
```

**Step 7:** Verify the data loaded correctly:
```bash
mysql -h $DB_ENDPOINT -u grcadmin -pGRC208awsgrclilly-2026 -e "USE grcdb; SHOW TABLES; SELECT COUNT(*) FROM compliance_summary;"
```

---

## 5. Testing and Validation

### Run Test Suite
In the VS Code terminal:
```powershell
# Ensure python is mapping the correct dependencies
pip install -r requirements.txt

# Run the test validations
python test_cases.py
```
<span style="color:red;">
**Layman Explanation:** This acts as your grading rubric. It runs a pre-written Python script to automatically inspect your AWS account and make sure everything is configured perfectly.<br>
**Project Objective:** Verifying your own infrastructure programmatically is standard DevOps practice. It ensures the capstone requirements were correctly met without having to manually click through 50 web pages.
</span>

### Manual Lambda Invocation Test
```powershell
# Test Lambda function
aws lambda invoke `
  --function-name grc-compliance-monitor `
  --region us-east-1 `
  response.json

Get-Content response.json
```
<span style="color:red;">
**Layman Explanation:** This manually presses the "Start" button on your Lambda script, asking it to run right now. It saves the answer to a file (`response.json`) and then prints the answer (`Get-Content`).<br>
**Project Objective:** Since the Lambda runs our GRC compliance check, we need to know the code doesn't crash on Day 1. This proves the logic works before we automate it.
</span>

A successful response looks like `"statusCode": 200` with compliance data in the body.

> [!WARNING]
> **If you see `"statusCode": 500` with `AccessDeniedException: config:DescribeComplianceByConfigRule`**, the Lambda role is missing Config permissions. Fix it by running:
> ```powershell
> aws iam attach-role-policy `
>   --role-name grc-lambda-role `
>   --policy-arn arn:aws:iam::aws:policy/AWSConfigUserAccess
> ```
> Then re-invoke the Lambda and check `response.json` again.

---

## 6. Post-Deployment Steps

Setup your EventBridge CloudWatch rules to trigger your Lambda hourly for automated assurance:

```powershell
# Create EventBridge rule
aws events put-rule `
  --name grc-compliance-check `
  --schedule-expression "rate(1 hour)" `
  --state ENABLED

# Append Lambda as target
aws events put-targets `
  --rule grc-compliance-check `
  --targets Id=1,Arn=arn:aws:lambda:us-east-1:630956767035:function:grc-compliance-monitor
```
<span style="color:red;">
**Layman Explanation:** EventBridge is a giant alarm clock. This command creates a 1-hour recurring alarm, and plugs our Lambda function into the alarm so it gets executed automatically.<br>
**Project Objective:** Continuous auditing is the core of this capstone. Instead of a human manually reviewing databases every quarter, this pipeline automatically validates compliance posture 24 times a day without human intervention.
</span>

---

## 7. Verify Deployment Status

Run these commands to confirm everything deployed successfully:

```powershell
# Check ALL CloudFormation stack statuses at once
aws cloudformation describe-stacks `
  --region us-east-1 `
  --query "Stacks[?starts_with(StackName,'grc-capstone')].{Name:StackName,Status:StackStatus,Created:CreationTime}" `
  --output table

# Check Network Stack specifically
aws cloudformation describe-stacks `
  --stack-name grc-capstone-network-stack `
  --region us-east-1 `
  --query "Stacks[0].StackStatus" `
  --output text

# Check Database Stack specifically
aws cloudformation describe-stacks `
  --stack-name grc-capstone-database-stack `
  --region us-east-1 `
  --query "Stacks[0].StackStatus" `
  --output text
```
<span style="color:red;">
**Layman Explanation:** This acts like a tracking number for a package. It queries AWS to see if your virtual networking and databases have successfully finished being built, and prints out their current status (like "CREATE_COMPLETE").<br>
**Project Objective:** You can't verify compliance on infrastructure that failed to deploy. Checking stack statuses ensures you are grading a fully operational environment.
</span>

You should see `CREATE_COMPLETE` for both stacks. If you see `ROLLBACK_COMPLETE` or `CREATE_FAILED`, check the error events:

```powershell
# View failure reasons
aws cloudformation describe-stack-events `
  --stack-name grc-capstone-database-stack `
  --region us-east-1 `
  --query "StackEvents[?ResourceStatus=='CREATE_FAILED'].{Resource:LogicalResourceId,Reason:ResourceStatusReason}" `
  --output table
```
<span style="color:red;">
**Layman Explanation:** If the 3D-Printer broke down halfway through, this command asks "What exactly caused the error?" and prints out the specific failure reason.<br>
**Project Objective:** Accelerates troubleshooting by pulling raw crash logs instead of guessing what failed in the cloud.
</span>

Also verify the individual resources that were deployed outside of CloudFormation:

```powershell
# Check Lambda function exists
aws lambda get-function `
  --function-name grc-compliance-monitor `
  --region us-east-1 `
  --query "Configuration.{Name:FunctionName,Runtime:Runtime,State:State}" `
  --output table

# Check AWS Config recorder is running
aws configservice describe-configuration-recorder-status `
  --region us-east-1 `
  --query "ConfigurationRecordersStatus[0].recording"

# Check EventBridge rule exists
aws events describe-rule `
  --name grc-compliance-check `
  --region us-east-1 `
  --query "{Name:Name,State:State,Schedule:ScheduleExpression}" `
  --output table
```
<span style="color:red;">
**Layman Explanation:** This is a roll-call check for the 3 individual manual parts we built: "Lambda, are you there?", "AWS Config, are you currently recording?", "EventBridge, are you scheduled?"<br>
**Project Objective:** Confirms the active security/monitoring pipeline is fully functional and running without errors.
</span>

---

## 8. Full Stack Teardown (Deletion)

When your assignment is complete, you **must** tear everything down to avoid charges. The database has `DeletionProtection` enabled, so CloudFormation will refuse to delete it unless you remove the lock first.

### 8.1 Remove Database Deletion Protection

```powershell
aws rds modify-db-instance `
  --db-instance-identifier grc-capstone-db `
  --no-deletion-protection `
  --apply-immediately `
  --region us-east-1
```
<span style="color:red;">
**Layman Explanation:** This turns off an administrative safety lock that prevents the database from ever being deleted.<br>
**Project Objective:** In real-world GRC, you never want your central audit database to be accidentally deleted, so we locked it. But to clean up the environment and stop AWS charges, we explicitly have to unlock it first.
</span>

### 8.2 Delete the CloudFormation Stacks

Delete them in **reverse order** (database first, then network):

```powershell
# Delete the Database Stack (takes ~10 mins)
aws cloudformation delete-stack `
  --stack-name grc-capstone-database-stack `
  --region us-east-1

# Wait for it to fully disappear
aws cloudformation wait stack-delete-complete `
  --stack-name grc-capstone-database-stack `
  --region us-east-1

# Delete the Network Stack (takes ~5 mins)
aws cloudformation delete-stack `
  --stack-name grc-capstone-network-stack `
  --region us-east-1

# Wait for it to fully disappear
aws cloudformation wait stack-delete-complete `
  --stack-name grc-capstone-network-stack `
  --region us-east-1
```
<span style="color:red;">
**Layman Explanation:** This tells the AWS 3D-Printer to go in reverse, dismantling every single network cable and database server it built earlier.<br>
**Project Objective:** Financial compliance is part of governance too! Tearing down expensive resources stops AWS from billing your account for cloud computers you are no longer actively using.
</span>

### 8.3 Verify Stacks Are Gone

```powershell
aws cloudformation describe-stacks `
  --region us-east-1 `
  --query "Stacks[?starts_with(StackName,'grc-capstone')].{Name:StackName,Status:StackStatus}" `
  --output table
```

If nothing shows up, both stacks are fully destroyed.

---

## 9. Manual Console Cleanup (Resources NOT Deleted by CloudFormation)

CloudFormation only deletes the resources it created. The following resources were created **manually via CLI** and must be cleaned up separately, either via terminal or the AWS Console.

### 9.1 Delete Lambda Function
```powershell
aws lambda delete-function `
  --function-name grc-compliance-monitor `
  --region us-east-1
```
<span style="color:red;">
**Layman Explanation:** Uninstalls and deletes your Python scanner script from the AWS servers.<br>
**Project Objective:** Avoids being charged for idle Lambda storage limits after the project is submitted.
</span>

### 9.2 Delete IAM Roles and Detach Policies
```powershell
# Detach policies from Lambda role, then delete the role
aws iam detach-role-policy `
  --role-name grc-lambda-role `
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

aws iam detach-role-policy `
  --role-name grc-lambda-role `
  --policy-arn arn:aws:iam::aws:policy/AWSConfigUserAccess

aws iam detach-role-policy `
  --role-name grc-lambda-role `
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

aws iam delete-role --role-name grc-lambda-role

# Detach policies from Config role, then delete the role
aws iam detach-role-policy `
  --role-name grc-config-role `
  --policy-arn "arn:aws:iam::aws:policy/service-role/AWS_ConfigRole"

aws iam delete-role --role-name grc-config-role
```
<span style="color:red;">
**Layman Explanation:** Security best practices say you must rip off the permission stickers before throwing away the ID badge. This strips the policies away and then deletes the ID badges entirely.<br>
**Project Objective:** Stray IAM policies left behind in an account represent a massive risk "surface area" for hackers. Governance requires full cleanup of unused administrative privileges.
</span>

### 9.3 Delete AWS Config Recorder and Delivery Channel
```powershell
# Stop the recorder first
aws configservice stop-configuration-recorder `
  --configuration-recorder-name grc-recorder `
  --region us-east-1

# Delete the delivery channel (must be deleted before the recorder)
aws configservice delete-delivery-channel `
  --delivery-channel-name grc-channel `
  --region us-east-1

# Delete the recorder
aws configservice delete-configuration-recorder `
  --configuration-recorder-name grc-recorder `
  --region us-east-1
```
<span style="color:red;">
**Layman Explanation:** This tells the security guard to stop recording, destroys the pipeline to the locker, and fires the guard.<br>
**Project Objective:** AWS Config charges an ongoing hourly fee for active recorders. This guarantees you won't get a surprise bill next month.
</span>

### 9.4 Delete EventBridge Rule and Targets
```powershell
# Remove the Lambda target first (rules with targets can't be deleted)
aws events remove-targets `
  --rule grc-compliance-check `
  --ids "1" `
  --region us-east-1

# Delete the rule
aws events delete-rule `
  --name grc-compliance-check `
  --region us-east-1
```
<span style="color:red;">
**Layman Explanation:** Unplugs the Lambda from the 1-hour alarm clock, then destroys the alarm clock.<br>
**Project Objective:** Leaving automated scripts running indefinitely creates "zombie" processes. This is a bad architecture practice and a waste of cloud resources.
</span>

### 9.5 Empty and Delete S3 Buckets Created Manually
S3 buckets **cannot be deleted until they are empty**. Run these commands:

```powershell
# Empty and delete the Config bucket
aws s3 rm s3://grc-config-bucket-630956767035 --recursive
aws s3 rb s3://grc-config-bucket-630956767035

# Empty and delete the CloudTrail bucket
aws s3 rm s3://grc-cloudtrail-logs-630956767035 --recursive
aws s3 rb s3://grc-cloudtrail-logs-630956767035
```
<span style="color:red;">
**Layman Explanation:** AWS refuses to delete a storage bucket if stuff is inside it. The `rm` command acts as a vacuum to aggressively delete all logs inside, and `rb` (remove bucket) destroys the bucket itself.<br>
**Project Objective:** S3 buckets charge for data storage. Purging the compliance logs securely deletes sensitive audit data when the environment is retired.
</span>

### 9.6 Delete CloudTrail
```powershell
aws cloudtrail delete-trail `
  --name grc-trail `
  --region us-east-1
```
<span style="color:red;">
**Layman Explanation:** Turns off the master administrative audit log.<br>
**Project Objective:** Similar to AWS Config, active CloudTrails incur costs for logging API events.
</span>

### 9.7 Final Console Verification (Manual Check)
Log into the **AWS Console** in your browser and manually verify these services have no leftover resources:

| Service | Where to Check | What to Look For |
|---------|---------------|-----------------|
| **CloudFormation** | CloudFormation > Stacks | No `grc-capstone-*` stacks |
| **RDS** | RDS > Databases | No `grc-capstone-db` instance |
| **S3** | S3 > Buckets | No `grc-capstone-*`, `grc-config-*`, or `grc-cloudtrail-*` buckets |
| **Lambda** | Lambda > Functions | No `grc-compliance-monitor` function |
| **IAM** | IAM > Roles | No `grc-lambda-role` or `grc-config-role` |
| **DynamoDB** | DynamoDB > Tables | No `grc-compliance-status`, `grc-risk-register`, or `grc-controls` tables |
| **KMS** | KMS > Customer managed keys | Schedule deletion for `grc-capstone-db-key` and `grc-capstone-s3-key` |
| **CloudWatch** | CloudWatch > Log Groups | Delete `/aws/lambda/grc-compliance-monitor` log group |
| **EventBridge** | EventBridge > Rules | No `grc-compliance-check` rule |
| **CloudTrail** | CloudTrail > Trails | No `grc-trail` trail |

> **Note on KMS Keys:** KMS keys cannot be instantly deleted. In the AWS Console, go to **KMS > Customer managed keys**, select the `grc-capstone-*` keys, click **Schedule key deletion**, and set the waiting period to the minimum (7 days). AWS will automatically purge them after that period.
