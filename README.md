# Cloudtrail Workshop 2021:
This small workshop walks a customer through the ability to alert on specific events that cloudtrail records and evaluate them in 2 ways:
1. Through AWS EventBridge + AWS SNS Email Notification
2. Through AWS Config + AWS SNS Email Notification

## Architectural Diagram:
![Architectural Diagram](/cloudtrail-workshop-2021.jpg)

## Steps to follow: 
1. Go into AWS IAM Console and create 3 AWS IAM Roles, call them:
    - `Authorized`, this role will have `s3-full-access` managed policy attached to it. Mark the `ARN` of this role in a notepad, you will use it later. (Typically, we would use much less permisive policy for S3 here)
    - `Unauthorized`, this role will have `s3-read-only` managed policy attached to it. Mark the `ARN` of this role in a notepad, you will use it later.
    - `MyConfigRole`, this role will have `AWSConfigRole` and `AmazonSNSFullAccess` managed policies attached to it. Mark the `ARN` of this role in a notepad, you will use it later. (Typically, we would use much less permisive policy for SNS here, [inspiration here](https://docs.aws.amazon.com/config/latest/developerguide/iamrole-permissions.html))
2. Create a Amazon S3 bucket and call it with unique name, for example `my-monitored-bucket-4839dna94`. Mark the `ARN` of this bucket in a notepad, you will use it later. When creating this bucket, set-up `Default Encryption` as `Enabled` and use the value `AWS Key Management Service key (SSE-KMS)` <details>![S3 Bucket Encryption Configuration](/step2.png)</details>
3. Create an Amazon SNS topic called `EncryptionDisabled` and set it to be `Standard` <details>![SNS Topic Configuration](/step3.png)</details>
4. Create a Amazon SNS Subscription of type `Email` and your email address, where you want to be receiving notifications. Once you create this subscription, head over to your email and confirm it, until you confirm it, no notifications will be sent there. <details>![SNS Subscription Configuration](/step4.png)</details>
5. Create an EventBridge Rule called `EncryptionDisabled`, set it to `Event Pattern`, then to `Pre-defined patern by service`, later select Service provier as `AWS`, Service name as `Simple Storage Service (S3)`, Event type to `Bucket Level Operations` and finally to a Specific operation(s) of `DeleteBucketEncryption`. As a target of this rule, we select `SNS Topic` and we pick the topic created in the previous step from the dropdown. <details>![EventBridge Rule Configuration](/step5.png)</details>
6. Open AWS Config Console, click on `Rules` and `Add Rule`. In the wizard, search for `s3-default-encryption-kms` from AWS Managed Rules. Click Next. On the next screen, name sure to set Resource Identifier to only specific bucket name that you created in the step 2. Once the rule is done, click on Rule `Actions` and `Re-evaluate`. This should yield `Compliant` status as this particular bucket is encrypted. <details>![Config Rule Configuration](/step6.png)</details> 
7. In AWS Console, go to `Settings` menu. There, click on `Edit` and turn-on `Stream configuration changes and notifications to an Amazon SNS topic.`. Then select a SNS topic we created in the step 3, called `EncryptionDisabled`. <details>![Config SNS Settings](/step7.png)</details> Make sure to also change AWS Config Role from `Use an existing AWS Config service-linked role` to `MyConfigRole` we created in the step 1.  <details>![Config Role Settings](/step7b.png)</details>
8. Open AWS CloudShell Console. Now you will try to assume the `Unauthorized` role by execuriting this command: `eval $(aws sts assume-role --role-arn ARN --role-session-name unauthorizedRole | jq -r '.Credentials | "export AWS_ACCESS_KEY_ID=\(.AccessKeyId)\nexport AWS_SECRET_ACCESS_KEY=\(.SecretAccessKey)\nexport AWS_SESSION_TOKEN=\(.SessionToken)\n"')`
    - once this role is assumed, let's verify the `assume-role` call by calling `aws sts get-caller-identity`, we should observe that now we act as unauthorized role we created. 
    - now, we will try to turn-off the encryption on the S3 bucket, let's try running  `aws s3 delete-bucket-encryption --bucket ARN`
    - what has happened? are we notified? can we see any change on our AWS Config Dashboard?
9. Now you will try to assume the `Authorized` role by execuriting this command: `eval $(aws sts assume-role --role-arn ARN --role-session-name unauthorizedRole | jq -r '.Credentials | "export AWS_ACCESS_KEY_ID=\(.AccessKeyId)\nexport AWS_SECRET_ACCESS_KEY=\(.SecretAccessKey)\nexport AWS_SESSION_TOKEN=\(.SessionToken)\n"')`
    - once this role is assumed, let's verify the `assume-role` call by calling `aws sts get-caller-identity`, we should observe that now we act as unauthorized role we created. 
    - now, we will try to turn-off the encryption on the S3 bucket, let's try running  `aws s3 delete-bucket-encryption --bucket ARN`
    - what has happened? are we notified? can we see any change on our AWS Config Dashboard?


## Shortcut
Deploy this entire architecture through AWS CloudFormation. Click here