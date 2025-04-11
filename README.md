# AWS Lambda Repository

## **Overview**  
This repository contains **independently developed and deployed AWS Lambda functions**, designed to work within [Adage: AWS Deployment Framework](https://github.com/tstrall/adage).  
- **Decoupled from Terraform** – Lambdas do not require Terraform to be updated when new services are added.  
- **Dynamically resolve dependencies** – Using AWS Parameter Store, Lambdas discover database endpoints, secrets, and other resources at runtime.  
- **Secure & auditable** – Secrets are retrieved from AWS Secrets Manager only when needed.  
- **Easily scalable** – Deploy multiple versions for feature branches or parallel environments.  

---

## **Repository Structure**
```
aws-lambda/
│── user-auth-service/
│   ├── app.py              # Lambda function code
│   ├── requirements.txt    # Dependencies
│   ├── config.json         # Default local configuration
│   ├── deploy.sh           # Deployment script
│── data-processing-service/
│   ├── app.py
│   ├── requirements.txt
│   ├── config.json
│   ├── deploy.sh
│── scripts/
│   ├── sync-config.sh      # Syncs Lambda-related configs to AWS Parameter Store
│── README.md
```

---

## **How It Works**
### **1️⃣ Lambdas Discover Infrastructure at Runtime**
Instead of **hardcoding infrastructure details**, Lambdas retrieve **runtime configurations** from AWS Parameter Store.  

For example, if `user-auth-service` needs access to an **Aurora-Postgres database**, it looks up:  
```
/aws/aurora-postgres/main-db/runtime ✅ (Contains DB host, port, security groups)
/aws/aurora-postgres/main-db/secret  ✅ (Contains DB credentials, retrieved securely)
```

### **2️⃣ Secrets Are Retrieved Securely**
Sensitive credentials (e.g., DB passwords, API keys) **are never stored in Parameter Store**.  
Instead, only **the ARN of the secret is stored**, and the Lambda retrieves the secret at runtime from AWS Secrets Manager.

---

## **Example: Configuring & Deploying a Lambda**
### **1️⃣ Define the Lambda Configuration**
Add a JSON entry to the **[`aws-config`](https://github.com/your-username/aws-config)** repository:
```json
{
  "database_nickname": "main-db",
  "sns_topic": "user-auth-notifications"
}
```

### **2️⃣ Deploy the Lambda**
Run the deploy script for `user-auth-service`:
```sh
cd user-auth-service
./deploy.sh
```

### **3️⃣ Lambda Resolves Dependencies at Runtime**
The Lambda **reads its dependencies from AWS Parameter Store**:
```python
import boto3
import json
import os

ssm = boto3.client("ssm")
secretsmanager = boto3.client("secretsmanager")

# 1️⃣ Retrieve database runtime details
db_config = json.loads(
    ssm.get_parameter(Name="/aws/aurora-postgres/main/runtime")["Parameter"]["Value"]
)

db_host = db_config["endpoint"]
db_port = db_config["port"]
db_user = db_config["username"]
secret_arn = db_config["secret_arn"]

# 2️⃣ Retrieve the database password securely
db_secret = json.loads(secretsmanager.get_secret_value(SecretId=secret_arn)["SecretString"])
db_password = db_secret["password"]

print(f"Connecting to {db_host}:{db_port} as {db_user}")
```
✅ **No hardcoded database URLs or passwords**—everything is dynamically resolved.

---

## **🔄 Supporting Multiple Versions & Environments**
Since each Lambda function **follows the same naming convention**, it’s easy to:
- Deploy multiple versions (`main`, `feature-x`, `staging`).
- Deploy new services **without modifying Terraform**.
- Support **temporary test environments**.

Example AWS Parameter Store entries for multiple Lambda versions:
```
/aws/lambda/user-auth-service/main/config ✅ (Main production instance)
/aws/lambda/user-auth-service/staging/config ✅ (Staging version)
/aws/lambda/user-auth-service/feature-x/config ✅ (Feature-specific deployment)
```

---

## **Security & Compliance**
✅ **Lambdas only access resources they need** – IAM roles follow **least privilege**.  
✅ **Secrets are retrieved securely from AWS Secrets Manager** – No plaintext credentials.  
✅ **Configurations are version-controlled via Git** – Full audit trail of changes.  
✅ **Works across multiple AWS accounts** – No hardcoded region or account-specific values.  

---

## Project Background

This repository is part of a broader open-source architecture I’ve developed to support configuration-driven AWS deployment.

While some of these ideas were shaped through years of professional experience and refinement, the implementations here are entirely original — built independently and outside the context of any prior employment.

For the full context and design principles behind this system, see [Adage: AWS Deployment Framework](https://github.com/tstrall/adage)

---

## ** Next Steps**
Want to implement this in your AWS environment? Here’s what to do next:  
1️⃣ **Fork this repo and customize the Lambda functions.**  
2️⃣ **Connect it to `aws-config` for dynamic configuration.**  
3️⃣ **Deploy and test!**  

📩 **Questions? Reach out or contribute!**  
This is an open-source approach, and improvements are always welcome.  

---

**Like this approach? Star the repo and follow for updates!**
