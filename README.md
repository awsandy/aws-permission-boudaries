## Using AWS Permission Boundaries

Can be applied to IAM User or Roles

A permissions boundary will define the maximum permissions that can be applied - regardless of what has been applied to the user or role


The policy below - is attached to the end user:

* It allows the user to perform some Lambda operations , S3 operations and read/list IAM
* It then also when creating a role
  * Ensures that the role naming convention is followed.
  * Ensures the Permission Boundary policy BoundaryForJenkins is selected when creating a role
  * Restricts the Roles that can be passed to Lambda to ones that conform to the naming convention.


*Policy: MinimalPermissionsForJenkins*

```
{
    "Version" : "2012-10-07",
    "Statement": [
        {
          "Sid":"LambdaOperations",
            "Effect": "Allow", 
            "Action": [
                "lambda:CreateFunction",
                "lambda:DeleteFunction",
                "lambda:UpdateFunction",
                "lambda:List*",
                "lambda:Get*",
                "lambda:Invoke*",
                "cloudformation:DescribeStack",
                "tag:getResources"
            ],
            "Resource": "*"    
        },
        {
        "Sid":"s3Operations",
            "Effect": "Allow", 
            "Action": [
                "s3:*"
            ],
            "Resource": "*"    
        },
        {
        "Sid":"IAMReadOnly",
            "Effect": "Allow", 
            "Action": [
                "iam:Get*",
                "iam:List*",
            ],
            "Resource": "*"    
        },
        {
        "Sid":"IAMRoleWithBoundary",
            "Effect": "Allow", 
            "Action": [
                "iam:CreateRole",
                "iam:DeleteRole",
                "iam:PutRolePolicy",
                "iam:DeleteRolePolicy",
                "iam:AttachRolePolicy",
                "iam:DetachRolePolicy"
            ],
            "Resource": "arn:aws:iam::111122223333:role/JenkinsDeployed-*",
            "Condition: {
                "StringEquals": {
                    "iam:PermissionsBoundary": "arn:aws:iam::111122223333:policy/BoundaryForJenkins"
                }
            }
        },
        {
            "Sid": "PassServerLessRole",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::111122223333:role/JenkinsDeployed-*"
        }
    ]
}
```

This is the boundary policy that must be attached to any Role that is created
Permission boundary never grants permissions it only narrows the scope of allowed permissions.

*Policy BoundaryForJenkins*

```
{
    "Version" : "2012-10-07",
    "Statement": [
        {
            "Sid": "CreateLogGroups",
            "Effect": "Allow",
            "Action": "logs:CreateLogsGroups",
            "Resource": "arn:aws:logs:eu-west-2:111122223333:*"
        },
        {
            "Sid": "CreateLogs",
            "Effect": "Allow",
            "Action": [
                "logs":"CreateLogStream",
                "logs":"PutLogEvent",
            "Resource": "arn:aws:logs:eu-west-2:111122223333:log-group:/aws/lambda/*"
        },
        "Sid":"s3ReadOnly",
            "Effect": "Allow", 
            "Action": [
                "s3:Get*",
                "s3:List*",
            ],
            "Resource": "arn:aws:s3:::serverless-bucket-proja*"    
        }
    ]
}

```

This above policy means that even if the user creates a Lambda role named "JenkinsDeployed-proja" with full admin rights, the permissions will be scoped down to those listed in the above BoundaryForJenkins policy.

### References:

https://www.youtube.com/watch?v=eVNvjQ0wr84

