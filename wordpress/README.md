
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

5. Route 53 domain registration and associated email (to validate ownership of domain). Note that: it is easy to set-up AWS WorkMail for this very purpose

6. An AWS Certificate Manager certificate in us-east-1 (CloudFront) containing all virtual host names of the domain(s) (e.g. website-test.info and *.website-test.info names)

6. Create a suitable website/apache/vhosts.conf file for the website in the core stack ConfigS3Bucket


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

