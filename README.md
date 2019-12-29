# Landing Zone

## Introduction

A starter AWS Landing Zone.

The AWS Landing Zone can be rolled out to one or more accounts. The delineation of responsibilities of AWS accounts is up to you. In some orgnisations accounts may be split up across applications/services and/or environments. In many cases there are likely to be separate accounts for the following responsibilities:

* AWS Organizations master account, which may also be used for consolidated billing
* Security, e.g. CloudTrail logging, Config etc.
* Shared Services, e.g. for single or multi-account Codepipeline, CodeBuild projects etc.

One account is designated the Shared Services account in this Landing Zone. In single-account configurations the account is shared with VPC, applications etc.

## Prequisites

Install AWS CLI, AWS SAM CLI and cfn-lint:

```bash
pip3 install --user --upgrade awscli cfn-lint
```

### Fork repository

Fork this GitHub repository to get started. Then clone the repository to your local system.

Make the necessary Landing Zone changes to the forked/cloned repository for your set-up.

### AWS Accounts

Consider using the following table in your own repository to record all of the AWS accounts that make up your account structure.

The following table details the AWS accounts.

| Account ID         | Acount alias           | Login URL                                                 | Root email                                        |
| ------------------ | ---------------------- | --------------------------------------------------------- | ------------------------------------------------- |
| 123456789012       | my-shared-services     | https://my-shared-services.signin.aws.amazon.com/console  | aws@my.com                                        |
| 234567890123       | development            | https://development.signin.aws.amazon.com/console         | aws@my.com                                        |
| 345678901234       | test                   | https://test.signin.aws.amazon.com/console                | aws@my.com                                        |
| 456789012345       | production             | https://production.signin.aws.amazon.com/console          | aws@my.com                                        |

A number of bootstrapping activities are required in the AWS accounts for Pipelines and Cloudformation deployments.

* Add Code Pipeline encryption key to the Shared Services account
* Add Code Pipeline Artifact S3 bucket to the Shared Services account
* Set-up single/cross-account Pipeline role in each environment AWS account (for describing stacks and creating Cloudformation change sets)
* Set-up single/cross-account Cloudformation role in each environment AWS account (for stack deployment)

These activities must be carried out with requisite privileges and credentials in each relevant AWS account.

### Pipeline Encryption Key & S3 Bucket

Can be carried out from AWS Management Console.

In the Shared Services AWS account create the Pipeline Encryption Key and S3 bucket via the AWS account's Management Console Cloudformation page. Navigate to the AWS Management Console [Cloudformation](https://console.aws.amazon.com/cloudformation/home) page and do the following:

* Click Create Stack
* Choose Upload template file and click Choose file; navigate to and pick the `bootstrap/shared-services-account-prereqs.yaml` file
* Click Next
* On the Parameters screen provide a Stack name. Suggestion: LZSharedServicesAccountPrereqs
* Fill out the AWS account ids of the Development, Test and Production accounts (may be the same account) (you can obtain the AWS account id from the [account Support page](https://console.aws.amazon.com/support/home))
* Click Next
* Click Next
* Under Capabilities check the "The following resource(s) require capabilities: [AWS::IAM::Role]" option
* Click Create stack

On successful creation of the stack, record the stack outputs SharedServicesAccountId, PipelineEncryptionKeyArn and PipelineArtifactBucketArn (Outputs tab) as they will be needed in the Cross-account IAM roles section.

### Cross-account IAM roles

Can be carried out from AWS Management Console.

For all environment AWS accounts create the cross-account IAM roles via each AWS account's Management Console Cloudformation page. Navigate to the AWS Management Console [Cloudformation](https://console.aws.amazon.com/cloudformation/home) page and do the following:

* Click Create Stack
* Choose Upload template file and click Choose file; navigate to and pick the `bootstrap/environment-account-prereqs.yaml` file
* Click Next
* On the Parameters screen provide a Stack name. Suggestion: LZEnvironmentAccountPrereqs
* On the Parameters screen also fill out the parameter pSharedServicesAccountId with the account id of the shared services account, i.e. the account you wish to run the Codepipeline from (you can obtain the account id from the AWS account [Support page](https://console.aws.amazon.com/support/home))
* Also fill out the AWS account ids of the Development, Test and Production accounts (may be the same account)
* Click Next
* Click Next
* Under Capabilities check the "The following resource(s) require capabilities: [AWS::IAM::Role]" option
* Click Create stack

## Core network rollout

Refer to the [Core network README](core/README.md) for instructions on how to build the core network in each AWS development, test and production account.

For 3 environments, e.g. development, test and production you will need to build the core network stack 3 times in your environment accounts.

Again, they may be same account.

## Examples

Refer to [EXAMPLES.md](EXAMPLES.md) for some sample additions to a landing zone.
