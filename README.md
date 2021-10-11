# Cloudtrail Workshop 2021:
This small workshop walks a customer through the ability to alert on specific events that cloudtrail records and evaluate them in 2 ways:
1. Through AWS EventBridge + AWS SNS Email Notification
2. Through AWS Config + AWS SNS Email Notification

## Architectural Diagram:
![Architectural Diagram](/cloudtrail-workshop-2021.jpg)

## Steps to follow: 
1. Go into AWS IAM Console and create 2 AWS IAM Roles, call them:
    - `Authorized`, this role will have `s3-full-access` managed policy attached to it. Mark the `ARN` of this role in a notepad, you will use it later.
    - `Unauthorized`, this role will have `s3-read-only` managed policy attached to it. Mark the `ARN` of this role in a notepad, you will use it later.
2. Create a Amazon S3 bucket and call it with unique name, for example `my-monitored-bucket-4839dna94`. Mark the `ARN` of this bucket in a notepad, you will use it later. When creating this bucket, set-up `Default Encryption` as `Enabled` and use the value `AWS Key Management Service key (SSE-KMS)` <details>![S3 Bucket Encryption Configuration](/step2.png)</details>
3. 
3. Open AWS CloudShell Console. Now you will try to assume the `Unauthorized` role by execuriting this command: `eval $(aws sts assume-role --role-arn ARN --role-session-name unauthorizedRole | jq -r '.Credentials | "export AWS_ACCESS_KEY_ID=\(.AccessKeyId)\nexport AWS_SECRET_ACCESS_KEY=\(.SecretAccessKey)\nexport AWS_SESSION_TOKEN=\(.SessionToken)\n"')`
    - once this role is assumed, let's verify the `assume-role` call by calling `aws sts get-caller-identity`, we should observe that now we act as unauthorized role we created. 
    - now, we will try to turn-off the encryption on the S3 bucket, let's try running  `aws s3 delete-bucket-encryption --bucket ARN`
    - what has happened? are we notified? can we see any change on our AWS Config Dashboard?
4. Now you will try to assume the `Authorized` role by execuriting this command: `eval $(aws sts assume-role --role-arn ARN --role-session-name unauthorizedRole | jq -r '.Credentials | "export AWS_ACCESS_KEY_ID=\(.AccessKeyId)\nexport AWS_SECRET_ACCESS_KEY=\(.SecretAccessKey)\nexport AWS_SESSION_TOKEN=\(.SessionToken)\n"')`
    - once this role is assumed, let's verify the `assume-role` call by calling `aws sts get-caller-identity`, we should observe that now we act as unauthorized role we created. 
    - now, we will try to turn-off the encryption on the S3 bucket, let's try running  `aws s3 delete-bucket-encryption --bucket ARN`
    - what has happened? are we notified? can we see any change on our AWS Config Dashboard?


## Shortcut
Deploy this entire architecture through AWS CloudFormation. Click here