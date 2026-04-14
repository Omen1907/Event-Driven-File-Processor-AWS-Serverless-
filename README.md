
# 📦 S3 Event-Driven File Processor (AWS Serverless)

## 📖 Introduction

This project demonstrates a fully serverless, event-driven workflow on AWS.

When a file is uploaded to an S3 bucket, it automatically triggers a Lambda function that:

- Sends an email notification via SNS
    
- Copies the uploaded file to a backup S3 bucket
    

This pattern is widely used in real-world systems for automation, monitoring, and data safety—without managing any servers.

---

## 🚨 Problem Statement

Build a system that reacts to file uploads in S3 and performs the following:

- 📧 Send an email notification on every upload
    
- 📁 Copy the uploaded file to a backup bucket
    

The email notification includes:

- Source bucket name
    
- AWS region
    
- File name and size
    
- Upload IP address
    
- Event timestamp
    

---

## 🛠️ AWS Services Used

- **Amazon S3** – Stores files and triggers events
    
- **AWS Lambda** – Processes events serverlessly
    
- **Amazon SNS** – Sends email notifications
    
- **Amazon CloudWatch** – Logs and monitoring
    

---

## 🧠 Key Concepts Practiced

- **Lambda Handler** – Entry point of execution
    
- **Event Object** – JSON payload from S3
    
- **Context Object** – Runtime metadata
    
- **Environment Variables** – Configuration management
    
- **Boto3 SDK** – AWS service interaction
    
- **Event-Driven Architecture** – Automatic execution on triggers
    

---

## 🏗️ Architecture Overview

```
S3 (PUT Object Event)
        ↓
   AWS Lambda
     ↙     ↘
 SNS Email   Copy to Backup S3
```

This architecture is fully serverless, scalable, and requires zero infrastructure management.

---

## ⚙️ Implementation Steps

### 1️⃣ Configure S3 Event Trigger

- Set up an S3 bucket to trigger Lambda on `ObjectCreated (PUT)` events.
    

---

### 2️⃣ Create SNS Topic

- Create an SNS topic
    
- Add and confirm email subscribers
    

---

### 3️⃣ Lambda Function Logic

The Lambda function:

- Extracts metadata from the S3 event
    
- Formats an email message
    
- Sends notification via SNS
    
- Copies the file to a backup bucket
    

---

## 💻 Lambda Code

```python
import os,json,boto3

s3 = boto3.client("s3")
sns = boto3.client("sns")

def lambda_handler(event,context):
    print(event)
    source_bucket = event['Records'][0]['s3']['bucket']['name']
    aws_region = event['Records'][0]['awsRegion']
    key_val = event['Records'][0]['s3']['object']['key']
    size_val = event['Records'][0]['s3']['object']['size']/1024
    ipAddress = event['Records'][0]['requestParameters']['sourceIPAddress']
    event_time = event['Records'][0]['eventTime']

    message = f"""Hi,
The Event time is: {event_time}
You are receiving this email because you are subscribed to this S3 event.
Source Bucket: {source_bucket}
AWS Region: {aws_region}
Uploaded Filename: {key_val} (Size: {size_val} KB)
Object uploaded from IP Address: {ipAddress}
"""

    backupBucket = os.environ['BACKUP_BUCKET_NAME']
    snsArn = os.environ['SNS_TOPIC_ARN']
    emailSubject = "S3EventTrigger-Notification"
    copy_source = {'Bucket' : source_bucket, 'Key' : key_val}

    # Send SNS notification
    sns.publish(
        TopicArn=snsArn,
        Message=message,
        Subject=emailSubject
    )

    # Copy file to backup bucket
    try:
        print("Copying the object from source to destination")
        s3.copy_object(
            Bucket=backupBucket,
            Key=key_val,
            CopySource=copy_source
        )
    except Exception as e:
        print(e)
        raise e
```

---

## 🔐 Environment Variables

|Key|Value|
|---|---|
|`BACKUP_BUCKET_NAME`|Your backup S3 bucket|
|`SNS_TOPIC_ARN`|Your SNS topic ARN|

---

## 🔑 IAM Permissions

The Lambda execution role requires:

### SNS Permissions

```json
"sns:Publish"
```

### S3 Permissions

```json
"s3:GetObject",
"s3:PutObject",
"s3:ListBucket"
```

---

## ▶️ How It Works

1. Upload a file to the source S3 bucket
    
2. S3 triggers the Lambda function
    
3. Lambda:
    
    - Extracts file details
        
    - Sends an SNS email
        
    - Copies the file to backup storage
        
4. Logs are stored in CloudWatch
    

---

## ✅ Verification

### 🔍 CloudWatch Logs

Check:

```
/aws/lambda/<your-lambda-name>
```

---

### 📁 Backup Bucket

- Confirm the file is copied successfully
    

---

### 📧 Email Notification

- Verify email contains:
    
    - File details
        
    - Upload metadata
        

---

## 📌 Outcome

This project demonstrates:

- Event-driven architecture in AWS
    
- Serverless automation workflows
    
- Secure configuration using environment variables
    
- Real-time notifications and backup strategies
    

---

## 🚀 What I Learned

- How S3 event notifications trigger Lambda
    
- How to integrate SNS with Lambda
    
- Handling AWS IAM permissions correctly
    
- Debugging with CloudWatch logs
    
- Building scalable serverless systems
    

---

## 🔮 Possible Improvements

- Handle multiple S3 records instead of one
    
- Add file type filtering
    
- Add versioning or unique naming in backup bucket
    
- Improve error handling and retries
    
- Integrate with API Gateway for extended workflows
    

---

## 📬 Final Thoughts

This project reflects a practical, production-style serverless pattern using AWS services. It shows how simple components can be combined to build powerful, automated systems.
    
- Turn this into a **portfolio-grade case study**
    
- Or help you write the **next AWS project (Auth system with Cognito)**
