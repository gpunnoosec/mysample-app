# File Upload Notification

## To set up CloudWatch Events notification

1. Create a target, such as an Amazon SNS topic or Lambda function, to invoke when the event you requested in AWS Storage Gateway is triggered.
2. Create a rule in the CloudWatch Events console to invoke targets based on an event in AWS Storage Gateway.
3. In the rule, create an event pattern for the event type. The notification is triggered when the event matches this rule pattern.
4. Select the target and configure the settings.

#### 1. Create a target for cloud watch events, SNS topic, invoked when event is triggered

**Amazon SNS > Topics > Create topic**

![SNS Create topic](https://i.imgur.com/y61SHWx.png)
**SNS Topic > Subscription > Protocol  > choose Email-json**
![SNSSubscription protocol](https://i.imgur.com/880vYTk.png)

**Access policy**

```
{
  "Version": "2012-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "__default_statement_ID",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": "SNS:Publish",
      "Resource": "arn:aws:sns:us-east-1:669760289733:a204503-access-policy-test",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:s3:::a204503-bucket"
        }
      }
    }
  ]
}
```

**Storage Gateway > File shares**
 a. Edit file share settings
![fileshare _edit file share settings](https://i.imgur.com/28992Pk.png)
a.1.  change to File upload notification from None to say 10 sec
![fileshare - change file upload notification](https://i.imgur.com/ONKqz5D.png)
b. Edit Email protocol to EMAIL-JASON
![Email protocol -email-json](https://i.imgur.com/TVu8W4A.png)
2. Create Rule & Select Target and Configure Event match
Step 1: Create Rule
![create rule and configure event match](https://i.imgur.com/fUbZr6s.png)
3.In the rule, create an event pattern for the event type
![rule - create event pattern](https://i.imgur.com/xHUTWRI.png)
4.Monitoring:
a.CloudWatch > Log group > /aws/storagegateway/
![cloudwatch-log group](https://i.imgur.com/63anSTy.png)
b.Click on Show metrics for the rule:
Graph shows Triggered rules 1 and Invocations 1
![monitoring rule metric](https://i.imgur.com/tMzGbIG.png)
**SNS target file upload Notification**
![SNS target email notification](https://i.imgur.com/CS5B792.png)
