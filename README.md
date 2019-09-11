# Background
A starter AWS Landing Zone.

The AWS Landing Zone can be rolled out to one or more accounts. The delineation of responsibilities of AWS accounts is up to you. In some orgnisations accounts may be split up across applications/services and/or environments. In many cases there are likely to be separate accounts for the following responsibilities:

* AWS Organizations master account, which may also be used for consolidated billing
* Security, e.g. CloudTrail logging, Config etc.
* Shared Services, e.g. for single or multi-account Codepipeline, CodeBuild projects etc.

One account is designated the Shared Services account in this Landing Zone. In single-account configurations the account is shared with VPC, applications etc.

# Bootstrap

## Fork repository

Fork this GitHub repository to get started. Then clone the repository to your local system.

Make the necessary Landing Zone changes to the forked/cloned repository for your set-up.

## AWS Accounts

Consider using the following table in your own repository to record all of the AWS accounts that make up your account structure.

The following table details the AWS accounts. 

| Account ID         | Acount alias           | Login URL                                                 | Root email                                        |
| ------------------ | ---------------------- | --------------------------------------------------------- | ------------------------------------------------- |
| 123456789012       | my-shared-services     | https://my-shared-services.signin.aws.amazon.com/console  | aws@my.com                                        |

A number of bootstrapping activities are required in the AWS accounts.

1. Set-up Account Administration role for StackSets & cross-account deployments in the Shared Services AWS account
2. Set-up Cloudformation Execution role for StackSets in each AWS account

These activities can be achieved via root in each AWS account.

## Shared Services cross-account IAM roles

For all AWS accounts create the Shared Services cross-account IAM roles via each AWS account's Management Console Cloudformation page.

There are two roles:

1. All accounts needs an execution role for Cloudformation (stack sets) - e.g. for Codepipeline execution
2. A cross-account role, which grants access to Cloudformation (cloudformation.amazonaws.com) to use the role above role from the shared services account.

The AWS default region is us-east-1 but you can choose to create the stack in any region as IAM is global.

For each AWS account navigate to the AWS Management Console [Cloudformation](https://console.aws.amazon.com/cloudformation/home) page and do the following:

* Click Create Stack
* Choose Upload template file and click Choose file; navigate to and pick the aws-lz/bootstrap/account-execution.yaml file
* Click Next
* On the Parameters screen provide a Stack name. Suggestion: LZCrossAccountExecution
* On the Parameters screen also fill out the parameter pSharedServicesAccountId with the account id of the shared services account, i.e. the account you wish to run the Codepipeline from (you can obtain the account id from the AWS account [Support page](https://console.aws.amazon.com/support/home))
* Click Next
* Click Next
* Under Capabilities check the "The following resource(s) require capabilities: [AWS::IAM::Role]" option
* Click Create stack

# Core network rollout

Refer to the [Core network README](core/README.md) for instructions on how to build the core network in each AWS environment account.

For 3 environments, e.g. development, test and production you will need to build the core network stack 3 times in your environment accounts.
