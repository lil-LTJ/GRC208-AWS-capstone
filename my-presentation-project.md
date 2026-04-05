# GRC208 AWS Capstone Project - 100% Submission Presentation Structure

**Student Name:** [Your Name]  
**Course:** GRC208  
**Instructor:** Aminu Idris - ICDFA  
**Date:** April 2026  

---

## Table of Contents
- [Slide 1: Account Setup Prerequisites & MFA](#slide-1-account-setup-prerequisites--mfa)
- [Slide 2: Phase 1 - Secure Infrastructure Deployment (IaC)](#slide-2-phase-1---secure-infrastructure-deployment-iac)
- [Slide 3: Phase 2 - Risk Management Storage (DynamoDB)](#slide-3-phase-2---risk-management-storage-dynamodb)
- [Slide 4: Phase 3 - Database Connectivity & Seeding](#slide-4-phase-3---database-connectivity--seeding)
- [Slide 5: Phase 4 - Automated Compliance Engine (Lambda)](#slide-5-phase-4---automated-compliance-engine-lambda)
- [Slide 6: Phase 5 - Continuous Audit Pipeline (EventBridge & AWS Config)](#slide-6-phase-5---continuous-audit-pipeline-eventbridge--aws-config)
- [Slide 7: Phase 6 - Automated Test Suite Validation](#slide-7-phase-6---automated-test-suite-validation)
- [Slide 8: Phase 7 - Resource Lifecycle Management & Teardown](#slide-8-phase-7---resource-lifecycle-management--teardown)
- [Conclusion](#conclusion)
- [Appendix: Supplementary Project Evidence](#appendix-supplementary-project-evidence)

---
## Slide 1: Account Setup Prerequisites & MFA
**Proof of Secure Account Foundation**

- **Where to Snapshot:** AWS Console > IAM > Security Credentials.
- **What to Snapshot:** Take a screenshot of the "Multi-factor authentication (MFA)" section showing a green checkmark/active MFA device.
- **Image:**  
  ![MFA Setup Secure Account Foundation](<./Snapshots form GRC208-AWS/MFA Setup Secure Account Foundation.JPG>)
- **Descriptive Explanation:** "This snapshot confirms the implementation of Multi-Factor Authentication (MFA) on the root/IAM account. In GRC (Governance, Risk, and Compliance), identity and access management is the first line of defense. By enabling MFA, the primary authentication vector is hardened against compromised credentials, fulfilling a core access control requirement."

---

## Slide 2: Phase 1 - Secure Infrastructure Deployment (IaC)
**Proof of Network and Database Provisioning**

- **Where to Snapshot:** AWS Console > CloudFormation > Stacks.
- **What to Snapshot:** A screenshot showing both `grc-capstone-network-stack` and `grc-capstone-database-stack` with a status of **CREATE_COMPLETE**.
- **Image:**  
  ![CloudFormation Infrastructure Stacks](<./Snapshots form GRC208-AWS/CloudFormation Infrastructure StacksJPG>)
- **Descriptive Explanation:** "This evidence demonstrates the successful deployment of the isolated cloud infrastructure using Infrastructure as Code (CloudFormation). Instead of manually clicking through menus, I used verifiable code to automatically generate a secure VPC network and an encrypted RDS database, eliminating human-error configuration drift."

---

## Slide 3: Phase 2 - Risk Management Storage (DynamoDB)
**Proof of NoSQL Risk Register Integration**

- **Where to Snapshot:** AWS Console > DynamoDB > Tables.
- **What to Snapshot:** A screenshot showing the active DynamoDB tables, specifically clicking into the `grc-capstone-RiskRegister` (or `grc-risk-register`) table showing it is "Active".
- **Image:**  
  ![DynamoDB Risk Register Table](<./Snapshots form GRC208-AWS/DynamoDB Risk Register Table.JPG>)
- **Descriptive Explanation:** "This snapshot proves the successful deployment of our DynamoDB Risk Register table. While traditional compliance controls stay in the RDS database, DynamoDB provides the high-speed, real-time NoSQL cache required for dynamic risk scoring matrices and rapid querying of vulnerability identifiers, enabling real-time status reporting."

---

## Slide 4: Phase 3 - Database Connectivity & Seeding
**Proof of Data Storage Initialization**

- **Where to Snapshot:** AWS CloudShell (or VS Code / MySQL Workbench).
- **What to Snapshot:** A screenshot of the terminal/interface after successfully executing the injection command, specifically showing the successful output of the query: `mysql -h $DB_ENDPOINT ... -e "USE grcdb; SELECT COUNT(*) FROM compliance_summary;"`
- **Image:**  
  ![Database Query Seeding Output](<./Snapshots form GRC208-AWS/Database Query Seeding Output.JPG>)
- **Descriptive Explanation:** "This snapshot validates that the RDS security groups were correctly configured to allow secure inbound connections. It proves that the relational database was successfully seeded with sample compliance frameworks (like ISO 27001, HIPAA, and NIST) and that the baseline compliance data is correctly structured and querying as expected."

---

## Slide 5: Phase 4 - Automated Compliance Engine (Lambda)
**Proof of Serverless Auditing Logistics**

- **Where to Snapshot:** VS Code Terminal.
- **What to Snapshot:** A screenshot of the terminal running the manual Lambda invocation command (`aws lambda invoke... response.json` and `Get-Content response.json`) showing a `{"statusCode": 200}` output.
- **Image:**  
  ![Lambda Function Execution Success](<./Snapshots form GRC208-AWS/Lambda Function Execution Success.JPG>)
- **Descriptive Explanation:** "This snapshot acts as proof of execution for the Python-based compliance monitor. By hosting the automation logic within an AWS Lambda function, the compliance scanner is serverless and isolated. The successful 200 response proves the function possesses the correct IAM permissions to securely scan vulnerabilities without failure."

---

## Slide 6: Phase 5 - Continuous Audit Pipeline (EventBridge & AWS Config)
**Proof of Uninterrupted System Auditing**

- **Where to Snapshot:** AWS Console > EventBridge > Rules.
- **What to Snapshot:** A screenshot of the `grc-compliance-check` rule showing its schedule expression (`rate(1 hour)`) and the Lambda function set as the target.
- **Image:**  
  ![EventBridge Hourly Schedule Rule](<./Snapshots form GRC208-AWS/EventBridge Hourly Schedule Rule.JPG>)
- **Descriptive Explanation:** "This snapshot validates the 'Continuous Monitoring' objective of modern GRC. By linking an EventBridge rule to our Lambda function, I have architected an automated compliance pipeline that independently triggers an audit 24 times a day. If any infrastructure parameters drift out of bounds, this system flags it within an hour."

---

## Slide 7: Phase 6 - Automated Test Suite Validation
**Proof of Technical Execution Excellence**

- **Where to Snapshot:** VS Code Terminal.
- **What to Snapshot:** A screenshot of the terminal after running `python test_cases.py`, showing that the assertions and functional checks have passed successfully.
- **Image:**  
  ![Automated Python Test Suite Success](<./Snapshots form GRC208-AWS/Automated Python Test Suite Success.JPG>)
- **Descriptive Explanation:** "This screenshot acts as the programmatic grading rubric. By running the pre-written unit suite (`test_cases.py`), it actively queries the AWS environment and asserts mathematically that all security controls, database connectivity, and backend infrastructure elements natively meet the Capstone's strict technical prerequisites seamlessly."

---

## Slide 8: Phase 7 - Resource Lifecycle Management & Teardown
**Proof of Secure End-of-Life Procedures**

- **Where to Snapshot:** AWS Console > Billing Dashboard > Budgets (Or CloudFormation empty stacks).
- **What to Snapshot:** A screenshot showing your active $10 Billing Alert budget, OR a screenshot of your CloudFormation stack page showing no `grc-capstone-*` stacks remain.
- **Image:**  
  ![Empty CloudFormation / Billing Budget](<./Snapshots form GRC208-AWS/IAM Budget.JPG>)
- **Descriptive Explanation:** "The final snapshot demonstrates overarching financial compliance and resource lifecycle governance. In cloud engineering, aggressively tearing down and dismantling unused infrastructure is just as important as building it; it prevents runaway overhead costs and minimizes the attack surface of unused resources. This confirms the environment was successfully retired."

---

## Conclusion
**Executive Summary of Capstone Achievements**

- **Key Takeaway 1:** Mastered the deployment of complex, multi-tier AWS environments strictly using Infrastructure as Code (IaC), eliminating risky manual console configurations.
- **Key Takeaway 2:** Successfully transformed stagnant compliance standards into an automated, living ecosystem by pairing AWS Config, EventBridge, and Lambda into a continuous auditing pipeline.
- **Key Takeaway 3:** Demonstrated comprehensive understanding of enterprise risk tracking, utilizing both relational structures (RMS MySQL) for steady-state data and NoSQL (DynamoDB) for dynamic risk calculations.
- **Final Statement:** "By completing this capstone, I have successfully demonstrated the ability to bridge complex cybersecurity requirements with modern DevOps automation. The resulting GRC platform proves that strictly adhering to governance does not stifle innovation; using AWS native services, security and agility can successfully coexist."

---

## Appendix: Supplementary Project Evidence
**Additional Screenshots & Architectural Documentation**

The following snapshots act as supplementary evidence corroborating the flawless deployment, orchestration, and teardown of the GRC capstone framework. They are included to ensure 100% technical traceability.

### 1. Verification of Network Foundation
**Snapshot:** `Check Network Stack`
- **Image:**  
  ![Check Network Stack](<./Snapshots form GRC208-AWS/Check Network Stack.JPG>)
- **Descriptive Explanation:** "This confirms the foundational network architecture stack is active, highlighting the proper setup of the custom VPC, secure subnets, and routing required for the Capstone."

### 2. Verification of Database Backend
**Snapshot:** `Check Database Stack`
- **Image:**  
  ![Check Database Stack](<./Snapshots form GRC208-AWS/Check Database Stack.JPG>)
- **Descriptive Explanation:** "A direct verification snapshot isolating the RDS database stack, proving that the secure backend storage is actively running and properly isolated from direct public interference."

### 3. Global Infrastructure Status
**Snapshot:** `Check ALL CloudFormation stack statuses`
- **Image:**  
  ![Check ALL CloudFormation stack statuses](<./Snapshots form GRC208-AWS/Check ALL CloudFormation stack statuses.JPG>)
- **Descriptive Explanation:** "This snapshot displays the AWS CLI verification checking the deployment status of all CloudFormation stacks simultaneously, confirming the successful overarching compilation of the environment."

### 4. Secondary Database Seeding Verification
**Snapshot:** `Database Query Seeding Output-2`
- **Image:**  
  ![Database Query Seeding Output-2](<./Snapshots form GRC208-AWS/Database Query Seeding Output-2.JPG>)
- **Descriptive Explanation:** "Secondary proof of the RDS database seeding phase. This showcases additional successful SQL query returns executed against the compliance data backend to prove row insertion worked effectively."

### 5. Secure Identity Posture
**Snapshot:** `MFA Setup Secure Account Foundation 2`
- **Image:**  
  ![MFA Setup Secure Account Foundation 2](<./Snapshots form GRC208-AWS/MFA Setup Secure Account Foundation 2.JPG>)
- **Descriptive Explanation:** "Additional documentation proving the robustness of the IAM root account security posture through supplementary configuration of Multi-Factor Authentication (MFA)."

### 6. CloudWatch Forensic Logging
**Snapshot:** `CloudWatch Log mgmt (console)`
- **Image:**  
  ![CloudWatch Log mgmt (console)](<./Snapshots form GRC208-AWS/CloudWatch Log mgmt (console).JPG>)
- **Descriptive Explanation:** "This snapshot from the AWS Console highlights CloudWatch logging execution. In GRC principles, maintaining immutable logs of system executions is required for forensic auditing and incident response."

### 7. Lambda Execution Role Verification
**Snapshot:** `Check Lambda function`
- **Image:**  
  ![Check Lambda function](<./Snapshots form GRC208-AWS/Check Lambda function.JPG>)
- **Descriptive Explanation:** "This captures the Lambda compliance monitor's successful configuration profile. It verifies that the serverless function was accurately deployed alongside its necessary IAM execution roles."

### 8. Graphical Lambda Execution Success
**Snapshot:** `Lambda Function Execution Success Console`
- **Image:**  
  ![Lambda Function Execution Success Console](<./Snapshots form GRC208-AWS/Lambda Function Execution Success Console.JPG>)
- **Descriptive Explanation:** "Visual verification directly from the AWS Lambda Console, demonstrating the successful graphical execution logs of the testing component without runtime errors or memory timeouts."

### 9. Config Recorder Initialisation
**Snapshot:** `Check AWS Config recorder`
- **Image:**  
  ![Check AWS Config recorder](<./Snapshots form GRC208-AWS/Check AWS Config recorder.JPG>)
- **Descriptive Explanation:** "This snapshot acts as proof that the core AWS Config recorder was successfully enabled and configured to record infrastructure drift, which is pivotal for the automated compliance tracking engine."

### 10. Centralized Evidence Storage
**Snapshot:** `S3 bucket`
- **Image:**  
  ![S3 bucket](<./Snapshots form GRC208-AWS/S3 bucket.JPG>)
- **Descriptive Explanation:** "A confirmation snapshot showing the successful deployment of the secure S3 bucket acting as the central storage repository for compliance evidence files and Config snapshots."

### 11. EventBridge Rule Configuration
**Snapshot:** `EventBridge Console`
- **Image:**  
  ![EventBridge Console](<./Snapshots form GRC208-AWS/EventBridge Console.JPG>)
- **Descriptive Explanation:** "A view from the EventBridge Console dashboard providing raw visual evidence that the automation bus is structurally active and correctly interpreting scheduling metrics."

### 12. Database Change Management History
**Snapshot:** `RDS Implementation History`
- **Image:**  
  ![RDS Implementation History](<./Snapshots form GRC208-AWS/RDS Implementation History.JPG>)
- **Descriptive Explanation:** "This snapshot captures the modification and lifecycle history of the RDS database, validating that proper change management controls were audited during the infrastructure's deployment."

### 13. Infrastructure Teardown: Databases
**Snapshot:** `CloudFormation Database deletion`
- **Image:**  
  ![CloudFormation Database deletion](<./Snapshots form GRC208-AWS/CloudFormation Database deletion.JPG>)
- **Descriptive Explanation:** "A critical piece of the project's financial teardown methodology, capturing the automated dismantling of the RDS database stack to ensure secure data hygiene upon project completion."

### 14. Infrastructure Teardown: Networking
**Snapshot:** `CloudFormation Network Stack deletion`
- **Image:**  
  ![CloudFormation Network Stack deletion](<./Snapshots form GRC208-AWS/CloudFormation Network Stack deletion.JPG>)
- **Descriptive Explanation:** "A final snapshot demonstrating the overarching end-of-life process. This verifies the VPC and supporting network subnets were successfully deleted alongside the database to prevent lingering resource costs."
