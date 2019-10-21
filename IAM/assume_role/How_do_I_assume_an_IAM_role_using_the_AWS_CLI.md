
# How do I assume an IAM role using the AWS CLI?

_Last updated: 2019-05-07_

I want to assume an Amazon Identity and Access Management (IAM) role using the AWS Command Line Interface (AWS CLI). How can I do this?  

## Resolution

Follow these instructions to assume an IAM role using the AWS CLI. In this example, the user will have read-only access to Amazon Elastic Compute Cloud (Amazon EC2) instances and permission to assume an IAM role.

**Create an IAM user that has permissions to assume roles**

1. [Create an IAM user using the AWS CLI](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html#id_users_create_cliwpsapi):

**Note:** Replace **Bob** with your IAM user name.  

```plainText
aws iam create-user --user-name Bob
```

2. Create the IAM policy that grants the permissions to **Bob** using the AWS CLI. You must create the JSON file that defines the IAM policy using your favorite text editor. This example uses vim, which is commonly used in Linux:

**Note:** Replace **example** with your own policy name, user name, role, JSON file name, profile name, and keys.  

```plainText
vim example-policy.json
```

3. The contents of the **example-policy.json** file should be similar to this:

```plainText
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:Describe*",
                "iam:ListRoles",
                "sts:AssumeRole"
            ],
            "Resource": "*"
        }
    ]
}
```

For more information about creating IAM policies, see [Creating IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html), [Example IAM Identity-Based Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html), and [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html).

**Create the IAM policy**

1. Use the **aws iam create-policy** command:

```plainText
aws iam create-policy --policy-name example-policy --policy-document file://example-policy.json
```

The aws iam create-policy command outputs several pieces of information, including the ARN (Amazon Resource Name) of the IAM policy:

```plainText
arn:aws:iam::123456789012:policy/example-policy
```

**Note:** Replace **123456789012** with your own account

2. Take note of the IAM policy ARN from the output, and attach the policy to **Bob** using the **attach-user-policy** command. Check to make sure that the attachment is in place using **list-attached-user-policies**:

```plainText
aws iam attach-user-policy --user-name Bob --policy-arn "arn:aws:iam::123456789012:policy/example-policy"
aws iam list-attached-user-policies --user-name Bob
```

**Create the JSON file that defines the trust relationship of the IAM role**

1. Create the JSON file that defines the trust relationship:

```plainText
vim example-role-trust-policy.json
```

2. The contents of the example-role-trust-policy.json file should be similar to this:

```plainText
{
    "Version": "2012-10-17",
    "Statement": {
        "Effect": "Allow",
        "Principal": { "AWS": "arn:aws:iam::123456789012:root" },
        "Action": "sts:AssumeRole"
    }
}
```

You can also restrict the trust relationship so that the IAM role can be assumed only by specific IAM users. You can do this by specifying principals similar to **arn:aws:iam::123456789012:user/example-username**. For more information, see [AWS JSON Policy Elements: Principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html).

**Create the IAM role and attach the policy**

Create an IAM role that can be assumed by **Bob** that has read-only access to Amazon Relational Database Service (Amazon RDS) instances. Because this IAM role is assumed by an IAM user, you must specify a principal that allows IAM users to assume that role. For example, a principal similar to **arn:aws:iam::123456789012:root** allows all IAM identities of the account to assume that role. For more information, see [Creating a Role to Delegate Permissions to an IAM User](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user.html).

1. Create the IAM role that has read-only access to Amazon RDS DB instances. Attach the IAM policies to your IAM role according to your security requirements.

The **aws iam create-role** command creates the IAM role and defines the trust relationship according to the contents of the JSON file. The **aws iam attach-role-policy** command attaches the AWS Managed Policy **AmazonRDSReadOnlyAccess** to the role. You can attach different policies (Managed Policies and Custom Policies) according to your security requirements. The **aws iam list-attached-role-policies** command shows the IAM policies that are attached to the IAM role **example-role**.

```plainText
aws iam create-role --role-name example-role --assume-role-policy-document file://example-role-trust-policy.json
aws iam attach-role-policy --role-name example-role --policy-arn "arn:aws:iam::aws:policy/AmazonRDSReadOnlyAccess"
aws iam list-attached-role-policies --role-name example-role
```

2. Check **Bob** to verify read-only access to EC2 instances, and if it's able to assume the **example-role**. Create access keys for **Bob** with this command:

```plainText
aws iam create-access-key --user-name Bob
```

The AWS CLI command outputs an access key ID and a secret access key—take note of these keys.

**Configure the access keys**

1. To configure the access keys, use either the default profile or a specific profile. To configure the default profile, run **aws configure**. To create a new specific profile, run **aws configure --profile example-profile-name**. In this example, the default profile is configured:

```plainText
aws configure
AWS Access Key ID [None]: ExampleAccessKeyID1
AWS Secret Access Key [None]: ExampleSecretKey1
Default region name [None]: eu-west-1
Default output format [None]: json
```

**Note:** For **Default region name**, specify your [AWS Region](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html).

**Verify that your AWS CLI commands are invoked and IAM user access**

1. Run the **aws sts get-caller-identity** command:

```plainText
aws sts get-caller-identity
```

The **aws sts get-caller-identity** command outputs three pieces of information including the ARN. It should show something similar to **arn:aws:iam::123456789012:user/Bob**, which verifies that the AWS CLI commands are invoked as **Bob**.

2. Confirm that the IAM user has read-only access to EC2 instances and no access to Amazon RDS DB instances by running these commands:

```plainText
aws ec2 describe-instances --query "Reservations[*].Instances[*].[VpcId, InstanceId, ImageId, InstanceType]"
aws rds describe-db-instances --query "DBInstances[*].[DBInstanceIdentifier, DBName, DBInstanceStatus, AvailabilityZone, DBInstanceClass]"
```

The **aws ec2 describe-instances** command should show you all the EC2 instances that are in the eu-west-1 Region. The **aws rds describe-db-instances** command must generate an access denied error message, because **Bob** doesn't have access to Amazon RDS.

**Assume the IAM role**

1. Get the ARN of the role by running this command:

```plainText
aws iam list-roles --query "Roles[?RoleName == 'example-role'].[RoleName, Arn]"
```

2. The command lists IAM roles but filters the output by role name. To assume the IAM role, run this command:

```plainText
aws sts assume-role --role-arn "arn:aws:iam::123456789012:role/example-role" --role-session-name AWSCLI-Session
```

The AWS CLI command outputs several pieces of information. Inside the credentials block you need the **AccessKeyId**, **SecretAccessKey**, and **SessionToken**. Take note of the timestamp of the expiration field. It is in the UTC time zone and indicates when the temporary credentials of the IAM role will expire. If the temporary credentials are expired, you must invoke the **sts:AssumeRole** API call again.  

**Note:** You can increase the maximum session duration expiration for temporary credentials for IAM roles using the [DurationSeconds](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithSAML.html#API_AssumeRoleWithSAML_RequestParameters) parameter.

**Create environment variables to assume the IAM role and verify access**

1. Create three environment variables to assume the IAM role. These environment variables are filled out with this output:

```plainText
export AWS_ACCESS_KEY_ID=ExampleAccessKeyID1
export AWS_SECRET_ACCESS_KEY=ExampleSecretKey1
export AWS_SESSION_TOKEN=ExampleSessionToken1
```

2. Verify that you assumed the IAM role by running this command:

```plainText
aws sts get-caller-identity
```

The AWS CLI command should output the ARN as **arn:aws:sts::123456789012:assumed-role/example-role/AWSCLI-Session** instead of **arn:aws:iam::123456789012:user/Bob**, which verifies that you assumed the **example-role**.

3. You created an IAM role with read-only access to Amazon RDS DB instances, but no access to EC2 instances. Verify by running these commands:

```plainText
aws ec2 describe-instances --query "Reservations[*].Instances[*].[VpcId, InstanceId, ImageId, InstanceType]"
aws rds describe-db-instances --query "DBInstances[*].[DBInstanceIdentifier, DBName, DBInstanceStatus, AvailabilityZone, DBInstanceClass]"
```

The **aws ec2 describe-instances** command should generate an access denied error message, and the **aws ec2 describe-instances** command should return the Amazon RDS DB instances. This verifies that the permissions assigned to the IAM role are working correctly.

4. To return to the IAM user, remove the environment variables:

```plainText
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN
aws sts get-caller-identity
```

The **unset** command removes the environment variables, and the **aws sts get-caller-identity** command verifies that you returned as **Bob**.

You can also use a role by creating a profile in the **~/.aws/config** file. For more information, see [Assuming an IAM Role in the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-role.html).
