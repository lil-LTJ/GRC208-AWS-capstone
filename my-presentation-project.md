# GRC208 AWS Capstone Project - 100% Submission Presentation Structure

**Student Name:** [Your Name]  
**Course:** GRC208  
**Instructor:** Aminu Idris - ICDFA  
**Date:** April 2026  

---

## Slide 1: Account Setup Prerequisites & MFA
**Proof of Secure Account Foundation**

- **Where to Snapshot:** AWS Console > IAM > Security Credentials.
- **What to Snapshot:** Take a screenshot of the "Multi-factor authentication (MFA)" section showing a green checkmark/active MFA device.
- **Image:**  
  `![MFA Setup Secure Account Foundation](./images/slide1_mfa.png)`
- **Descriptive Explanation:** "This snapshot confirms the implementation of Multi-Factor Authentication (MFA) on the root/IAM account. In GRC (Governance, Risk, and Compliance), identity and access management is the first line of defense. By enabling MFA, the primary authentication vector is hardened against compromised credentials, fulfilling a core access control requirement."

---

## Slide 2: Phase 1 - Secure Infrastructure Deployment (IaC)
**Proof of Network and Database Provisioning**

- **Where to Snapshot:** AWS Console > CloudFormation > Stacks.
- **What to Snapshot:** A screenshot showing both `grc-capstone-network-stack` and `grc-capstone-database-stack` with a status of **CREATE_COMPLETE**.
- **Image:**  
  `![CloudFormation Infrastructure Stacks](./images/slide2_cfn_stacks.png)`
- **Descriptive Explanation:** "This evidence demonstrates the successful deployment of the isolated cloud infrastructure using Infrastructure as Code (CloudFormation). Instead of manually clicking through menus, I used verifiable code to automatically generate a secure VPC network and an encrypted RDS database, eliminating human-error configuration drift."

---

## Slide 3: Phase 2 - Risk Management Storage (DynamoDB)
**Proof of NoSQL Risk Register Integration**

- **Where to Snapshot:** AWS Console > DynamoDB > Tables.
- **What to Snapshot:** A screenshot showing the active DynamoDB tables, specifically clicking into the `grc-capstone-RiskRegister` (or `grc-risk-register`) table showing it is "Active".
- **Image:**  
  `![DynamoDB Risk Register Table](./images/slide3_dynamodb.png)`
- **Descriptive Explanation:** "This snapshot proves the successful deployment of our DynamoDB Risk Register table. While traditional compliance controls stay in the RDS database, DynamoDB provides the high-speed, real-time NoSQL cache required for dynamic risk scoring matrices and rapid querying of vulnerability identifiers, enabling real-time status reporting."

---

## Slide 4: Phase 3 - Database Connectivity & Seeding
**Proof of Data Storage Initialization**

- **Where to Snapshot:** AWS CloudShell (or VS Code / MySQL Workbench).
- **What to Snapshot:** A screenshot of the terminal/interface after successfully executing the injection command, specifically showing the successful output of the query: `mysql -h $DB_ENDPOINT ... -e "USE grcdb; SELECT COUNT(*) FROM compliance_summary;"`
- **Image:**  
  `![Database Query Seeding Output](./images/slide4_db_seeding.png)`
- **Descriptive Explanation:** "This snapshot validates that the RDS security groups were correctly configured to allow secure inbound connections. It proves that the relational database was successfully seeded with sample compliance frameworks (like ISO 27001, HIPAA, and NIST) and that the baseline compliance data is correctly structured and querying as expected."

---

## Slide 5: Phase 4 - Automated Compliance Engine (Lambda)
**Proof of Serverless Auditing Logistics**

- **Where to Snapshot:** VS Code Terminal.
- **What to Snapshot:** A screenshot of the terminal running the manual Lambda invocation command (`aws lambda invoke... response.json` and `Get-Content response.json`) showing a `{"statusCode": 200}` output.
- **Image:**  
  `![Lambda Function Execution Success](./images/slide5_lambda_test.png)`
- **Descriptive Explanation:** "This snapshot acts as proof of execution for the Python-based compliance monitor. By hosting the automation logic within an AWS Lambda function, the compliance scanner is serverless and isolated. The successful 200 response proves the function possesses the correct IAM permissions to securely scan vulnerabilities without failure."

---

## Slide 6: Phase 5 - Continuous Audit Pipeline (EventBridge & AWS Config)
**Proof of Uninterrupted System Auditing**

- **Where to Snapshot:** AWS Console > EventBridge > Rules.
- **What to Snapshot:** A screenshot of the `grc-compliance-check` rule showing its schedule expression (`rate(1 hour)`) and the Lambda function set as the target.
- **Image:**  
  `![EventBridge Hourly Schedule Rule](./images/slide6_eventbridge.png)`
- **Descriptive Explanation:** "This snapshot validates the 'Continuous Monitoring' objective of modern GRC. By linking an EventBridge rule to our Lambda function, I have architected an automated compliance pipeline that independently triggers an audit 24 times a day. If any infrastructure parameters drift out of bounds, this system flags it within an hour."

---

## Slide 7: Phase 6 - Automated Test Suite Validation
**Proof of Technical Execution Excellence**

- **Where to Snapshot:** VS Code Terminal.
- **What to Snapshot:** A screenshot of the terminal after running `python test_cases.py`, showing that the assertions and functional checks have passed successfully.
- **Image:**  
  `![Automated Python Test Suite Success](./images/slide7_test_suite.png)`
- **Descriptive Explanation:** "This screenshot acts as the programmatic grading rubric. By running the pre-written unit suite (`test_cases.py`), it actively queries the AWS environment and asserts mathematically that all security controls, database connectivity, and backend infrastructure elements natively meet the Capstone's strict technical prerequisites seamlessly."

---

## Slide 8: Phase 7 - Resource Lifecycle Management & Teardown
**Proof of Secure End-of-Life Procedures**

- **Where to Snapshot:** AWS Console > Billing Dashboard > Budgets (Or CloudFormation empty stacks).
- **What to Snapshot:** A screenshot showing your active $10 Billing Alert budget, OR a screenshot of your CloudFormation stack page showing no `grc-capstone-*` stacks remain.
- **Image:**  
  `![Empty CloudFormation / Billing Budget](./images/slide8_teardown.png)`
- **Descriptive Explanation:** "The final snapshot demonstrates overarching financial compliance and resource lifecycle governance. In cloud engineering, aggressively tearing down and dismantling unused infrastructure is just as important as building it; it prevents runaway overhead costs and minimizes the attack surface of unused resources. This confirms the environment was successfully retired."

---

## Conclusion
**Executive Summary of Capstone Achievements**

- **Key Takeaway 1:** Mastered the deployment of complex, multi-tier AWS environments strictly using Infrastructure as Code (IaC), eliminating risky manual console configurations.
- **Key Takeaway 2:** Successfully transformed stagnant compliance standards into an automated, living ecosystem by pairing AWS Config, EventBridge, and Lambda into a continuous auditing pipeline.
- **Key Takeaway 3:** Demonstrated comprehensive understanding of enterprise risk tracking, utilizing both relational structures (RMS MySQL) for steady-state data and NoSQL (DynamoDB) for dynamic risk calculations.
- **Final Statement:** "By completing this capstone, I have successfully demonstrated the ability to bridge complex cybersecurity requirements with modern DevOps automation. The resulting GRC platform proves that strictly adhering to governance does not stifle innovation; using AWS native services, security and agility can successfully coexist."
