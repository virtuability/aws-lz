
# Core Cloudformation template


## Environment variables for build

Some common environment variables to set in order for commands below to find the template and config files.

    TMPL_DIR=$HOME/dev/git/cf-templates/core/template
    CNF_DIR=$HOME/dev/git/cf-templates/core/config
    STACK_NAME=core-test

## Core

Commands to validate the template and create, update etc. the stack. 

    aws cloudformation validate-template --template-body file://$TMPL_DIR/core.yaml

    aws cloudformation create-stack --stack-name $STACK_NAME --template-body file://$TMPL_DIR/core.yaml --parameters file://$CNF_DIR/$STACK_NAME-parameters.json --capabilities CAPABILITY_IAM

    aws cloudformation update-stack --stack-name $STACK_NAME --template-body file://$TMPL_DIR/core.yaml --parameters file://$CNF_DIR/$STACK_NAME-parameters.json --capabilities CAPABILITY_IAM

    aws cloudformation list-stacks --stack-status-filter CREATE_IN_PROGRESS CREATE_COMPLETE UPDATE_IN_PROGRESS UPDATE_COMPLETE DELETE_IN_PROGRESS ROLLBACK_COMPLETE ROLLBACK_IN_PROGRESS

    aws cloudformation delete-stack --stack-name $STACK_NAME

## OpenVPN Client configuration

The utility script /usr/bin/build-client.sh in the NAT VM can create the OpenVPN client configuration as an OVPN file.

Simply run the script as root with an arbitrary client name, e.g. "client1", as parameter:

    /usr/bin/build-client.sh client1

Once complete, you may find the configuration OVPN file in /etc/openvpn/easy-rsa/client1.ovpn.

Copy the file to the OpenVPN client system and import the OVPN file (with Windows client: Import file...)

