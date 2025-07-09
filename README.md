# 🔐 AWS Secrets Access Monitoring System

A robust, real-time AWS security monitoring pipeline that sends instant alerts when your sensitive secrets are accessed.

> ✅ Track API access to secrets
> 📊 Detect unauthorized behavior
> 📩 Get instant email alerts
> 💼 Designed for Cloud Security, DevOps, and Cloud Engineers

-----

## 🚀 What You’ll Build

A real-time detection system that alerts you whenever a secret stored in **AWS Secrets Manager** is accessed, using:

  * **AWS Secrets Manager**
  * **AWS CloudTrail**
  * **Amazon CloudWatch Logs, Metrics & Alarms**
  * **Amazon SNS (Simple Notification Service)**

-----

## 🧠 Architecture Diagram
![AWS Secrets Monitoring Architecture](https://github.com/FanoyG/aws-secrets-monitoring-system/blob/main/diagram-export-7-9-2025-1_18_15-PM.svg)

-----

## 📦 Stack Overview

| Component               | Purpose                                       |
| :---------------------- | :-------------------------------------------- |
| **AWS Secrets Manager** | Securely store secrets (e.g., DB credentials) |
| **AWS CloudTrail** | Track all `GetSecretValue` events             |
| **CloudWatch Logs** | Ingest CloudTrail logs                        |
| **CloudWatch Metric** | Detect secret access using filters            |
| **CloudWatch Alarm** | Trigger alert on threshold breach             |
| **Amazon SNS** | Send email notifications                      |

-----

## 🛠️ Step-by-Step Guide

### 1️⃣ Create and Store a Secret

1.  Open **AWS Secrets Manager** → `Store a new secret`
2.  Select: *Other type of secret*
3.  Enter:
      * **Key:** `db_password`
      * **Value:** `MyTest@123`
4.  **Secret Name:** `FanoyDBSecret`
5.  Disable automatic rotation

> **Pro Tip:**
> Use meaningful secret names and never store plaintext secrets in your code.

-----

### 2️⃣ Enable CloudTrail

1.  Go to **AWS CloudTrail** → `Create Trail`
2.  **Trail name:** `FanoyTrail`
3.  Enable for all regions
4.  Enable management events (Read/Write)
5.  Store logs in a new S3 bucket: `fanoy-cloudtrail-logs`

**(Optional CLI Command):**

```bash
aws cloudtrail create-trail --name FanoyTrail --s3-bucket-name fanoy-cloudtrail-logs --is-multi-region-trail
```

> **Pro Tip:**
> Enable CloudTrail for all regions to cover activity everywhere.

-----

### 3️⃣ Simulate Secret Access

**AWS CLI Command:**

```bash
aws secretsmanager get-secret-value --secret-id FanoyDBSecret --profile SecMonitorUser
```

> **Pro Tip:**
> Test with a dedicated IAM user to validate monitoring without exposing production credentials.

-----

### 4️⃣ Create CloudWatch Metric Filter

1.  Go to **CloudWatch** → Log Groups → Find your CloudTrail Log Group

2.  Click **Create Metric Filter**

3.  Use this pattern:

    ```json
    { $.eventName = "GetSecretValue" }
    ```

4.  **Metric Name:** `SecretsAccessed`

5.  **Namespace:** `SecurityMonitoring`

> **Pro Tip:**
> Monitor specific secrets or users by extending the pattern, e.g.:
> `{ $.eventName = "GetSecretValue" && $.requestParameters.secretId = "FanoyDBSecret" }`

-----

### 5️⃣ Create Alarm for Access Detection

1.  Go to **CloudWatch** → Alarms → Create Alarm
2.  Select `SecurityMonitoring/SecretsAccessed`
3.  **Threshold:** ≥ 1 in 5 minutes
4.  **Alarm Name:** `SecretsMonitorAlert`

> **Pro Tip:**
> Set thresholds and periods based on your environment’s typical access patterns.

-----

### 6️⃣ Set Up SNS Email Alerts

1.  Go to **SNS** → Topics → Create Topic `SecretAccessAlerts`
2.  Add email subscription (e.g., `you@example.com`)
3.  Confirm the subscription via email
4.  Attach this SNS topic to your CloudWatch alarm

> **Pro Tip:**
> SNS can notify multiple emails, SMS, Lambda, or even Slack via webhook.

-----

### 7️⃣ Test the System

**Option 1: Real Access**

```bash
aws secretsmanager get-secret-value --secret-id FanoyDBSecret
```

**Option 2: Manual Alarm Trigger**

```bash
aws cloudwatch set-alarm-state \
--alarm-name "SecretsMonitorAlert" \
--state-value ALARM \
--state-reason "Manual test trigger for security monitoring system"
```

-----

## 🟢 Expected Output

  * CloudTrail logs the event
  * CloudWatch filter detects it
  * Alarm triggers
  * SNS sends email alert

**Sample Email Notification:**

> ALARM: "SecretsMonitorAlert" has entered the ALARM state.

-----

## 💡 Additional Tips & Pro Guidance

  * **IAM Best Practices:** Grant only necessary permissions to users and roles accessing secrets.
  * **Cost Management:** Monitor CloudWatch and SNS usage to avoid unexpected charges.
  * **Compliance:** Retain CloudTrail logs as per your organization’s policy.
  * **Automation:** Use Infrastructure as Code (e.g., Terraform, AWS CDK) for repeatable deployments.
  * **Alert Fatigue:** Tune filters and thresholds to minimize false positives.

-----

## 🎯 Use Cases

  * Cloud Security Monitoring
  * Detect data exfiltration attempts
  * Compliance & Audit
  * IAM Behavioral Alerts

-----

## 🧠 What You Learn

  * Monitoring `GetSecretValue` access via CloudTrail
  * Using CloudWatch Metric Filters to create custom alerts
  * Creating real-time SNS notifications from alarms

-----

## 📌 Future Enhancements

  * Add automatic rotation with Lambda
  * Monitor specific IAM users/roles
  * Integrate with Slack or Microsoft Teams
  * Add a dashboard in CloudWatch

-----

# 🧹 AWS Cleanup Guide – Secret Monitor Project

This guide helps you **delete everything step-by-step** after testing your AWS Secret Monitoring setup. Use this to avoid charges. ✅

-----

## 🔐 What You Created (Before Cleanup)

  * **SNS Topic** → to send email alerts 📧
  * **CloudWatch Alarm** → to detect secret access 🚨
  * **Metric Filter** → to count secret access logs 📈
  * **CloudTrail** → to log every AWS API call 📋
  * **S3 Bucket** → to store CloudTrail logs 🪣
  * **(Optional) IAM Role** → to give CloudWatch permission 🔐

-----

## ✅ Cleanup Steps (Do in this order)

### 1️⃣ SNS – Delete Email Subscriptions & Topics

**Check subscriptions:**

```bash
aws sns list-subscriptions
```

If it says `"SubscriptionArn": "Deleted"` — you're good\! Otherwise, delete with:

```bash
aws sns unsubscribe --subscription-arn <your-subscription-arn>
```

**Delete the Topic (Optional):**

```bash
aws sns delete-topic --topic-arn arn:aws:sns:ap-south-1:<your-account-id>:SecurityAlarms
```

-----

### 2️⃣ CloudWatch – Delete Alarm, Metric Filter & Log Group

**Check for alarms:**

```bash
aws cloudwatch describe-alarms
```

If empty, you're done. Otherwise, delete:

```bash
aws cloudwatch delete-alarms --alarm-names SecretsMonitorAlert
```

**Check for metric filters:**

```bash
aws logs describe-metric-filters
```

If not empty:

```bash
aws logs delete-metric-filter \
--log-group-name <log-group-name> \
--filter-name <filter-name>
```

**Check for log groups:**

```bash
aws logs describe-log-groups
```

To delete:

```bash
aws logs delete-log-group --log-group-name <log-group-name>
```

-----

### 3️⃣ CloudTrail – Delete Trail & S3 Bucket

**Check for active trails:**

```bash
aws cloudtrail describe-trails
```

If empty, you're done. Otherwise:

```bash
aws cloudtrail delete-trail --name <your-trail-name>
```

**Check for leftover S3 log bucket:**

```bash
aws s3 ls
```

**Delete logs:**

```bash
aws s3 rm s3://your-bucket-name --recursive
```

**Then delete the bucket:**

```bash
aws s3api delete-bucket --bucket your-bucket-name
```

-----

### 4️⃣ IAM – (Optional) Delete Custom Role

If you created a custom role, delete with:

```bash
aws iam delete-role --role-name <role-name>
```

-----

### 💡 Final Tip

Always check your billing dashboard: [https://console.aws.amazon.com/billing](https://console.aws.amazon.com/billing)

-----

## 🧠 Summary

  * ✅ SNS: Subscriptions deleted
  * ✅ CloudWatch: Alarms, logs, filters deleted
  * ✅ CloudTrail: Trail & S3 cleaned
  * ✅ IAM: No roles to delete (unless you made one)

> 📦 This cleanup guide helps you avoid surprise charges and keeps your AWS account clean\!

-----

## 📜 License

MIT License — use it, learn from it, build your own.

-----

## ✍️ Author

Built by Fanoy – Python Dev & Future Cloud Engineer ☁️🐍 Let’s connect on LinkedIn\! *(insert your profile link)*

-----

## 📚 Resources

  * [Get the full step-by-step guide](https://www.google.com/search?q=https://www.youtube.com/watch%3Fv%3DwEtYJBSsgks%26list%3DPPSV)

-----
