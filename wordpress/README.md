
# WordPress Website Cloudformation template

This CloudFormation template provides a WordPress website configuration, which includes Web Server Autoscaling with the website stored on EFS.

# Build

The instructions below assume that the AWS CLI is used; but it's also possible to build with the AWS Console.

## Prerequisites

1. Spin up the core stack first

2. The following environment variables will need to be set correctly for template and configuration locations before running the commands in the following section (adjust as necessary):

    TMPL_DIR=$HOME/dev/git/cf-templates/wordpress/template
    CNF_DIR=$HOME/dev/git/cf-templates/wordpress/config
    STACK_NAME=website-test
    AWS_DEFAULT_REGION=eu-west-1

3. The necessary administrative permissions to create the stack

4. Creation or adjustment of parameters as required in the parameter file cf-templates/wordpress/config/$STACK_NAME-$AWS_DEFAULT_REGION.json

5. Copy the update_security_groups.py lambda zip file from cf-templates/wordpress/lambda/updater.zip to the core s3 bucket (as /website/lambda/updater.zip), e.g.

     aws s3 cp wordpress/lambda/update_security_groups.zip s3://core-test-configs3bucket-xxxxxxxxxxxx/wordpress/lambda/update_security_groups.zip

  Note: The lambda code was sourced from the AWS repository: https://github.com/awslabs/aws-cloudfront-samples

6. Route 53 domain registration and associated email (to validate ownership of domain). Note that it is easy to set-up AWS WorkMail for this very purpose

7. An AWS Certificate Manager certificate in us-east-1 (CloudFront) containing all virtual host names of the domain(s) (e.g. website-test.info and *.website-test.info names)


## Stack Creation

The following AWS CLI commands are used to validate the template and create, update etc. the stack:

    aws cloudformation validate-template --template-body file://$TMPL_DIR/wp.yaml

    aws cloudformation create-stack --stack-name $STACK_NAME --template-body file://$TMPL_DIR/wp.yaml --parameters file://$CNF_DIR/$STACK_NAME-$AWS_DEFAULT_REGION.json --capabilities CAPABILITY_IAM

    aws cloudformation update-stack --stack-name $STACK_NAME --template-body file://$TMPL_DIR/wp.yaml --parameters file://$CNF_DIR/$STACK_NAME-$AWS_DEFAULT_REGION.json --capabilities CAPABILITY_IAM

    aws cloudformation list-stacks --stack-status-filter CREATE_IN_PROGRESS CREATE_COMPLETE UPDATE_IN_PROGRESS UPDATE_COMPLETE DELETE_IN_PROGRESS ROLLBACK_COMPLETE ROLLBACK_IN_PROGRESS

    aws cloudformation delete-stack --stack-name $STACK_NAME

### Change Set Validation

One would be encouraged to use the change sets to validate changes before actually making them.

    aws cloudformation create-change-set  --stack-name $STACK_NAME --change-set-name $STACK_NAME-1 --template-body file://$TMPL_DIR/wp.yaml --parameters file://$CNF_DIR/$STACK_NAME-$AWS_DEFAULT_REGION.json --change-set-type UPDATE --capabilities CAPABILITY_IAM

    aws cloudformation execute-change-set --stack-name $STACK_NAME --change-set-name $STACK_NAME-1

    aws cloudformation delete-change-set --stack-name $STACK_NAME --change-set-name $STACK_NAME-1

## WordPress configuration

