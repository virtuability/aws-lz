# Examples

This document provides examples of common AWS account bootstrap scenarios.

## Create IAM users with scripted MFA automation

In many instances it's necessary to create IAM users to manage AWS accounts. It's always a good idea to secure IAM users with MFA and to take a tiered approach to user privileges (MFA vs. non-MFA).

This section details setting up a sample IAM "admin" user with two sets of policies:

* IAM user self-service policy, which enables the user to manage MFA and password changes
* Administrator privileges provided that the user is MFA-authenticated

The section also details how to set-up the sample user with MFA credentials and password change via the AWS CLI and supporting tools (like oathtool for MFA key generation). In effect this is the equivalent of setting up MFA via the AWS Console.

First, let's create the user. This assumes that valid AWS credentials already exist in the shell, e.g. from the root user during account bootstrap.

```bash
GIT_DIR=$HOME/dev/git/virtuability/aws-lz
DEPLOY_APP=iam-bootstrap
AWS_DEFAULT_REGION=eu-west-1
AWS_ACCOUNT_ID=`aws sts get-caller-identity | jq -r '.Account'`

cfn-lint $GIT_DIR/iam/$DEPLOY_APP.yaml

aws cloudformation validate-template \
  --template-body file://$GIT_DIR/iam/iam-bootstrap.yaml

# Create the Cloudformation stack
aws cloudformation create-stack \
  --stack-name $DEPLOY_APP \
  --template-body file://$GIT_DIR/iam/iam-bootstrap.yaml \
  --capabilities CAPABILITY_NAMED_IAM

# To update the Cloudformation stack later on:
aws cloudformation update-stack \
  --stack-name $DEPLOY_APP \
  --template-body file://$GIT_DIR/iam/iam-bootstrap.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

Create an MFA device for the `admin` user:

```bash
MFA_SERIAL_NO=`aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name adminmfa \
  --bootstrap-method Base32StringSeed \
  --outfile adminmfa.txt | jq -r ".VirtualMFADevice.SerialNumber"`
```

Install command line oauthtool to generate MFA/TOTP codes on behalf of the `admin` user:

```bash
sudo yum install oathtool
```

Enable and associate MFA device with the `admin` user (using 2 authentication codes for synchronisation and authorisation):

```bash
AUTH_CODE1=$(oathtool -b --totp $(cat adminmfa.txt)) ; sleep 30 ; AUTH_CODE2=$(oathtool -b --totp $(cat adminmfa.txt))

aws iam enable-mfa-device \
  --user-name admin \
  --serial-number $MFA_SERIAL_NO \
  --authentication-code1 $AUTH_CODE1 \
  --authentication-code2 $AUTH_CODE2
```

Set-up the `admin` user with an access key and secret - credentials further protected with MFA:

```bash
aws iam create-access-key \
  --user-name admin \
  | jq -r ".AccessKey | \"[admin]\naws_access_key_id = \" + .AccessKeyId + \"\naws_secret_access_key = \" + .SecretAccessKey + \"\n\"" >>$HOME/.aws/credentials

cat <<-EOF >>$HOME/.aws/config
[profile adminmfa]
output = json
region = eu-west-1
source_profile = admin
mfa_serial = $MFA_SERIAL_NO
EOF
```

Get a set of admin MFA-protected temporary credentials:

```bash
aws sts get-session-token \
  --duration-seconds 14400 \
  --serial-number $MFA_SERIAL_NO \
  --token-code $(oathtool -b --totp $(cat adminmfa.txt)) \
  --profile admin \
  | jq -r ".Credentials | \"export AWS_ACCESS_KEY_ID=\" + .AccessKeyId + \"\nexport AWS_SECRET_ACCESS_KEY=\" + .SecretAccessKey + \"\nexport AWS_SESSION_TOKEN=\" + .SessionToken + \"\n\""
```

Open new shell and run the exports with the admin MFA-protected temporary credentials:

```bash
# export AWS_ACCESS_KEY_ID=...
# export AWS_SECRET_ACCESS_KEY=...

```

Now try this set of credentials in the new shell by listing all S3 buckets:

```bash
aws s3 ls
```
