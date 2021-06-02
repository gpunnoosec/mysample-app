# IHN Data Services -Storage - CAB for AWS FSx
George Punnoose
May 27,2021


## Overview
### Testing : 
1. Cross Account Backup of FSx in same region
3. Testing Restore: From backup to primary region
4. Creating  a new FSx file system in remote region

#### Pattern 1 -FSx instance in a single AZ

#### Pattern 2 -FSx instance with HA -With AWS CRAB

### Cross-account Backup pre-requisites
1. Source AWS account and destination account must belong to the same AWS Organization.
TR-enterprise-ro-role![tr-enterprise-ro-role-Screenshot 2021-05-27 170901](/assets/tr-enterprise-ro-role-Screenshot%202021-05-27%20170901.png) 
tr-enterprise-AWS-org-ID![TR-enterprise AWS org IDScreenshot 2021-05-27 163445](https://i.imgur.com/zO7sMVk.png)
2. This AWS Organization must have trusted access enabled for the AWS Backup service.
AWS>>My account> Settings
cross-account backup allow permission![cross-account-bkp-Screenshot 2021-05-27 165849](/assets/cross-account-bkp-Screenshot%202021-05-27%20165849.png)
3. The source AWS account Amazon FSx file system must be created using a customer-managed KMS CMK.

```
Region: US-East-1
Source Account : 669760289733
Source Vault: a204503-fsx-bkpvault-destgp
Destination Account: 496598440491
Destination Backup Vault Name: a204503-gp-vault-cab
FSx Filesystem Name: a204503-fsx-cmk-gp
CMK  KMS Key ID : 
arn:aws:kms:us-east-1:669760289733:key/6c06ed55-6b1c-4370-a887-12a562a4fdbc
Key Policy for CMK – alias: cmk-fsx-testgp  is appended at the end.
```
KMS > Customer managed keys >Create key : Step1 Configure key
![create CMK key](https://i.imgur.com/3uTn3Mb.png)
KMS > Customer managed keys > Create key : Step 2 Add labels
![kms_cmk_add-labels](https://i.imgur.com/YrvYvpK.png)
KMS > Customer managed keys >Create key : Step 3 Define key administrative permissions
![KMS_CMK_create-key-step3_definekeyadmin permissions](https://i.imgur.com/HmhwprT.png) 
4.  The customer-managed KMS CMK used to create the source Amazon FSx file system must be    shared with the destination AWS account.
![KMS_cmk_createkey_define-key-usage_other-AWS-accounts](https://i.imgur.com/AmOZrYF.png)
5.  The destination AWS account AWS Backup vault must be created using a “Customer-managed KMS CMK”.
![destination vault in target account-496598440491 ARN](https://i.imgur.com/LQ2Eg2Z.png)
6.  The *destination AWS account AWS Backup vault* must have an 
    a.access policy that “allows CopyIntoBackupVault” to the AWS Organization. 
    b.From the AWS Backup Console, select the backup vault 
    c.And in the Access policy section select Add permissions >> Allow access to a Backup vault from organization.
**allow access** ![add permission to Backup vault from AWS Orga Screenshot 2021-05-24 103126](/assets/add%20permission%20to%20Backup%20vault%20from%20AWS%20Orga%20Screenshot%202021-05-24%20103126_ghu83hebd.png)



**Access Policy** Permissions at Destination vault  “Allow” at AWS Organization to copy into BackupVault 
tr-enterprise-AWS-org-ID![TR-enterprise AWS org IDScreenshot 2021-05-27 163445](https://i.imgur.com/zO7sMVk.png)
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "5b78c404-01e4-4f8e-8ed8-9b95aea4b60a",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::669760289733:root"
            },
            "Action": "backup:CopyIntoBackupVault",
            "Resource": "*"
        },
        {
            "Sid": "5b78c404-01e4-4f8e-8ed8-9b95aea4b1154",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "backup:CopyIntoBackupVault",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:PrincipalOrgID": "o-3ljayxintq"
                }
            }
        }
    ]
}
```
Key Policy for Custom Managed Key : alias -cmk-fsk-testgp
```
Key ID: 6c06ed55-6b1c-4370-a887-12a562a4fdbc
arn:aws:kms:us-east-1:669760289733:key/6c06ed55-6b1c-4370-a887-12a562a4fdbc

{
    "Version": "2012-10-17",
    "Id": "key-consolepolicy-3",
    "Statement": [
        {
            "Sid": "Enable IAM User Permissions",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::669760289733:root"
            },
            "Action": "kms:*",
            "Resource": "*"
        },
        {
            "Sid": "Allow access for Key Administrators",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::669760289733:root"
            },
            "Action": [
                "kms:Create*",
                "kms:Describe*",
                "kms:Enable*",
                "kms:List*",
                "kms:Put*",
                "kms:Update*",
                "kms:Revoke*",
                "kms:Disable*",
                "kms:Get*",
                "kms:Delete*",
                "kms:TagResource",
                "kms:UntagResource",
                "kms:ScheduleKeyDeletion",
                "kms:CancelKeyDeletion"
            ],
            "Resource": "*"
        },
        {
            "Sid": "Allow use of the key",
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "arn:aws:iam::142227596713:root",
                    "arn:aws:iam::669760289733:root",
                    "arn:aws:iam::496598440491:root"
                ]
            },
            "Action": [
                "kms:Encrypt",
                "kms:Decrypt",
                "kms:ReEncrypt*",
                "kms:GenerateDataKey*",
                "kms:DescribeKey"
            ],
            "Resource": "*"
        },
        {
            "Sid": "Allow attachment of persistent resources",
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "arn:aws:iam::669760289733:root",
                    "arn:aws:iam::142227596713:root",
                    "arn:aws:iam::669760289733:role/human-role/204503-PowerUser",
                    "arn:aws:iam::496598440491:root"
                ]
            },
            "Action": [
                "kms:CreateGrant",
                "kms:ListGrants",
                "kms:RevokeGrant"
            ],
            "Resource": "*",
            "Condition": {
                "Bool": {
                    "kms:GrantIsForAWSResource": "true"
                }
            }
        }
    ]
}
```
&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
**Steps for an on-demand backup:**
a.	From the source AWS account, create an on-demand backup of the Amazon FSx file system with a backup vault that was created with either an AWS managed KMS CMK or a customer-managed KMS CMK. 
b.	Once the backup is completed, select the backup and backup vault and select Actions >> Copy.
c.	Select the AWS Region to where you want to copy the backup.
d.	Select Copy to another account’s vault and paste in the ARN of the destination AWS account AWS Backup vault.
e.	It is NOT necessary to “Allow” access to the destination AWS account to copy back to the source AWS account. 
f.	Select Copy
**Steps to create a backup plan:**
a.	Select build a new plan
b.	Select the source backup vault that was created with either an AWS managed KMS CMK or a customer-managed KMS CMK.
c.	Select the AWS Region to where you want to copy the backup.
d.	Select Copy to another account’s vault and paste in the ARN of the destination AWS account AWS Backup vault.
e.	Select Create plan
f.	Select the backup plan and select Assign resources.
g.	Select the Amazon FSx file system ID using resource ID or using tags

Test  | FSx	| Source Backup Vault |	Destination Backup Vault |	Result
------|-----|---------------------|--------------------------|-------
1 |	AWS managed CMK |	AWS managed CMK	| AWS managed CMK |	Failure
2 |	AWS managed CMK	|Customer-managed CMK |	AWS managed CMK |	Failure
3 |	Customer-managed CMK |	AWS managed CMK |	Customer-managed CMK |	Success
4 |	Customer-managed CM |	Customer-managed CMK |	Customer-managed CMK |	Success
