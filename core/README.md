
# Core Cloudformation template

GIT_DIR=$HOME/dev/git/cf-templates/core/template
CNF_DIR=$HOME/dev/git/cf-templates/core/config
STACK_NAME=core-test

## Core

aws cloudformation validate-template --template-body file://$GIT_DIR/templates/core.yaml

aws cloudformation create-stack --stack-name $STACK_NAME --template-body file://$GIT_DIR/templates/core.yaml --parameters file://$CNF_DIR/$STACK_NAME-parameters.json --capabilities CAPABILITY_IAM

aws cloudformation update-stack --stack-name $STACK_NAME --template-body file://$GIT_DIR/templates/core.yaml --parameters file://$CNF_DIR/$STACK_NAME-parameters.json --capabilities CAPABILITY_IAM

aws cloudformation list-stacks --stack-status-filter CREATE_IN_PROGRESS CREATE_COMPLETE UPDATE_IN_PROGRESS UPDATE_COMPLETE DELETE_IN_PROGRESS ROLLBACK_COMPLETE ROLLBACK_IN_PROGRESS

aws cloudformation delete-stack --stack-name $STACK_NAME


