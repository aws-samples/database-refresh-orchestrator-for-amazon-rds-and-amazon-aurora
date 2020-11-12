README.md

## Database refresh orchestrator for Amazon RDS and Amazon Aurora

This repository contains the resources required to build the solution described in the AWS Database Blog post https://github.com/aws-samples/aws-systems-manager-database-voice-commands/edit/master/README.md.

The package **awssoldb-orchestrator-pkg-cloudformation.zip** represents the solution (CloudFormation templates, Lambda function's code and sample sql-scripts).

The package **awssoldb-orchestrator-launch.zip** contains the code to launch (manually, in this demo) the orchestrator.

## Installation guide

### Pre-requirements

Here the list of the pre-requirements you need to satisfy before deploy and test the solution:

1. Choose the region where you want to deploy the solution. The demo has been tested in Virginia [us-east-1] but you can change it if necessary, by considering the availability of the services involved

1. Download the two packages **awssoldb-orchestrator-pkg-cloudformation.zip** and **awssoldb-orchestrator-launch.zip**

1. The computer from which you will perform the two tests below will need Python (this tutorial was tested with Python 3)

1. Create an S3 bucket in the same region where you will deploy the solution. The bucket must have the following structures:

	* A directory named **templates** which contains all the CloudFormation templates (from the package **awssoldb-orchestrator-pkg-cloudformation.zip**)
	* A directory named **functions** which contains the code of all the Lambda functions associated with this project (from the package **awssoldb-orchestrator-pkg-cloudformation.zip**)

1. Create an EC2 key pair in the same region where you will deploy the solution (optional, if you already have one)

1. Create an SNS Topic with an E-mail subscriber in the same region you will deploy the solution (optional, if you already have one)

This tutorial uses the DEFAULT VPC.

### Deploy the infrastructure with AWS CloudFormation

Choose the region where you want to deploy your infrastructure and then submit the **awssoldb_global.template** to CloudFormation.

The creation should take around 25 minutes (most of the time is taken by the RDS database instance and the Aurora cluster).

### Post-deploy steps

After the successful creation of the infrastructure by CloudFormation, please do the following:

1. Modify the two refresh files **db-app2-auposinstd.json** and **db-app1-mysqlinstd.json** (from the package **awssoldb-orchestrator-launch.zip**):
	* In the "restore" element you must change the value of the key "secgrpids":

		**[original value]** "secgrpids": "CHANGE_ME"

		**[new value]** "secgrpids": "sg-xxx" (where "sg-xxx" is the ID of the VPC Security Group "RDSSecGrp-awssoldb")

	* In the "sendmsg" element you must change the value of the key "topicarn":

		**[original value]** "topicarn": "arn:aws:sns:CHANGE_ME"

		**[new value]** "topicarn": "arn:aws:sns:<region_code>:<account_id>:<sns_topic_name>"

## Test the solution

### Test 1: Cloning an existing Aurora cluster using Aurora Fast-cloning

1. Execute the Step Functions state machine "state-machine-awssol" using the Python script **launch_refresh.py** (from the package **awssoldb-orchestrator-launch.zip**)

	* $ cd awssoldb-orchestrator-launch
	* $ python3 launch_refresh.py auposinstd app2 arn:aws:states:<region>:<account_id>:stateMachine:state-machine-awssol <region>

1. Monitor the status of the cloning operation using the Step Functions dashboard and the RDS dashboard

1. At the end of the cloning operation you will find a new entry in the DynamoDB table "dbalignment-awssol" and you will get an e-mail with a confirm about the cloning just completed

### Test 2: Cloning an existing RDS database instance through a Point-In-Time-Restore

TBD

## Clean up

To delete everything you need to:

1. Delete the new Aurora cluster created during "Test 1"

1. Delete the new RDS database instance created during "Test 2"

1. Delete the parent CloudFormation stack, the one associated with the **awssoldb_global.template** template. The deletion of all the stacks should take around 10 minutes


## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

