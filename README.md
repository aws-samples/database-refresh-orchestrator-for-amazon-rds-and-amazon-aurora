README.md

## Database refresh orchestrator for Amazon RDS and Amazon Aurora

This repository contains the resources required to build the solution described in the AWS Database Blog post https://github.com/aws-samples/aws-systems-manager-database-voice-commands/edit/master/README.md.

The package **awssoldb-orchestrator-pkg-cloudformation.zip** represents the solution (CloudFormation templates, Lambda function's code and sample sql-scripts).

The package **awssoldb-orchestrator-launch.zip** contains the code to launch (manually, in this demo) the orchestrator.

## Installation guide

### Pre-requirements

Here the list of the pre-requirements you need to satisfy before deploy and test the solution:

1. Choose the region where you want to deploy the solution. The demo has been tested in Virginia [us-east-1] but you can change it if necessary, by considering the availability of the services involved

1. The demo has been tested with a Linux Client (in particularly with an AWS Cloud9 environment with Amazon Linux) but you can choose the type of client you want (i.e. a generic EC2 instance, a local computer, etc...). The client needs to have the following requirements:

	* Python installed (this demo has been tested with Python3)
	* AWS SDK for Python (boto3) installed
	* AWS CLI installed and configured
	* IAM credentials configured (at the CLI level or at the role level)

1. Download the two packages **awssoldb-orchestrator-pkg-cloudformation.zip** and **awssoldb-orchestrator-launch.zip** and unzip them. The two packages need to be staged on the client you will use for the demo:

	* $ mkdir awssoldb
	* $ cd ./awssoldb
	* # Download the two packages...
	* $ unzip awssoldb-orchestrator-launch.zip
	* $ unzip awssoldb-orchestrator-pkg-cloudformation.zip

1. Create an S3 bucket in the same region where you will deploy the solution. The bucket must have the following structures:

	* A directory named **templates** which contains all the CloudFormation templates (from the package **awssoldb-orchestrator-pkg-cloudformation.zip**), so the files with extension *.template
	* A directory named **functions** which contains the code of all the Lambda functions associated with this project (from the package **awssoldb-orchestrator-pkg-cloudformation.zip**), so the files with extension *.zip
	* A directory path named **sqlscripts/rdsmysql/mysqlinstp/** which contains the SQL scripts **pre-reqs.sql** and **final-check.sql** (from the package **awssoldb-orchestrator-pkg-cloudformation.zip**)
	* A directory path named **sqlscripts/rdsmysql/mysqlinstd/** which contains the SQL scripts **01script.sql** and **02script.sql** (from the package **awssoldb-orchestrator-pkg-cloudformation.zip**)

1. Create an EC2 key pair in the same region where you will deploy the solution (optional, if you already have one)

1. Create an SNS Topic with an E-mail subscriber in the same region you will deploy the solution (optional, if you already have one)

This tutorial uses the DEFAULT VPC.

### Deploy the infrastructure with AWS CloudFormation 

Choose the region where you want to deploy your infrastructure and then submit the **awssoldb_global.template** to CloudFormation:

1. Open the CloudFormation dashboard in the region where you will deploy the solution

1. Create a new stack with new resources

1. Create template in Designer

1. Copy and paste the content of the template **awssoldb_global.template**

1. Specify the following information in the "Specify stack details" page:

	* **Stack name**
	* **paramDBPwd**: Master user password for the RDS instances that will use for the testing the solution
	* **paramKeyPairForEc2**: EC2 key pair for the EC2 instance used to run post-refresh SQL scripts
	* **paramVPCId**: The ID of a VPC in the current region (this demo has been tested in the DEFAULT VPC)
	* **paramS3bucket**: S3 bucket name created above
	* **paramSNSTopic**: SNS Topic created above
	* **paramSubnetId1ForLambda**: First subnet Id for the Lambda functions used to run SQL scripts (the function will be deployed within a VPC)
	* **paramSubnetId2ForLambda**: Second subnet Id for the Lambda functions used to run SQL scripts (the function will be deployed within a VPC)
	* **paramSubnetIdForEc2**: Subnet Id for the EC2 instance used to run post-refresh SQL scripts

The creation should take around 25 minutes (most of the time is taken by the RDS database instance and the Aurora cluster).

### Post-deploy steps

After the successful creation of the infrastructure by CloudFormation, please do the following on the client where you have downloaded the two packages **awssoldb-orchestrator-pkg-cloudformation.zip** and **awssoldb-orchestrator-launch.zip**: 

1. Modify the refresh files **db-app1-mysqlinstd.json** (from the package **awssoldb-orchestrator-launch.zip**):
	* In the "restore" element you must change the value of the key "secgrpids":

		**[original value]** "secgrpids": "CHANGE_ME"

		**[new value]** "secgrpids": "sg-xxx" (where "sg-xxx" is the ID of the VPC Security Group "RDSSecGrp-awssoldb")

	* In the "sendmsg" element you must change the value of the key "topicarn":

		**[original value]** "topicarn": "CHANGE_ME"

		**[new value]** "topicarn": "arn:aws:sns:<region_code>:<account_id>:<sns_topic_name"

	* In the "runscripts" element you must change the value of the key "bucketname":

		**[original value]** "bucketname": "CHANGE_ME"

		**[new value]** "bucketname": "xxx" (where "xxx" is the name of the bucket you created above)

	* In the "runscripts" element you must change the value of the key "check.bucketname":

		**[original value]** "bucketname": "CHANGE_ME"

		**[new value]** "bucketname": "xxx" (where "xxx" is the name of the bucket you created above)

	* In the "fixtags" element you must change the value of the key "dbarn":

		**[original value]** "dbarn": "arn:aws:rds:CHANGE_ME:CHANGE_ME:db:mysqlinstd"

		**[new value]** "dbarn": "arn:aws:rds:xxx:yyy:db:mysqlinstd" (where "xxx" is the current region and "yyy" is the current AWS Account ID)

	* In the "fixtags" element you must change the value of the key "check.dbarn":

		**[original value]** "dbarn": "arn:aws:rds:CHANGE_ME:CHANGE_ME:db:mysqlinstd"

		**[new value]** "dbarn": "arn:aws:rds:xxx:yyy:db:mysqlinstd" (where "xxx" is the current region and "yyy" is the current AWS Account ID)

1. Modify the refresh files **db-app2-auposinstd.json** (from the package **awssoldb-orchestrator-launch.zip**):
	* In the "restore" element you must change the value of the key "secgrpids":

		**[original value]** "secgrpids": "CHANGE_ME"

		**[new value]** "secgrpids": "sg-xxx" (where "sg-xxx" is the ID of the VPC Security Group "RDSSecGrp-awssoldb")

	* In the "sendmsg" element you must change the value of the key "topicarn":

		**[original value]** "topicarn": "CHANGE_ME"

		**[new value]** "topicarn": "arn:aws:sns:<region_code>:<account_id>:<sns_topic_name"

	* In the "fixtags" element you must change the value of the key "cluarn":

		**[original value]** "cluarn": "arn:aws:rds:CHANGE_ME:CHANGE_ME:cluster:auposclud"

		**[new value]** "cluarn": "arn:aws:rds:xxx:yyy:cluster:auposclud" (where "xxx" is the current region and "yyy" is the current AWS Account ID)

	* In the "fixtags" element you must change the value of the key "check.cluarn":

		**[original value]** "cluarn": "arn:aws:rds:CHANGE_ME:CHANGE_ME:cluster:auposclud"

		**[new value]** "cluarn": "arn:aws:rds:xxx:yyy:cluster:auposclud" (where "xxx" is the current region and "yyy" is the current AWS Account ID)

	* In the "fixtags" element you must change the value of the key "dbarn":

		**[original value]** "dbarn": "arn:aws:rds:CHANGE_ME:CHANGE_ME:db:auposinstd"

		**[new value]** "dbarn": "arn:aws:rds:xxx:yyy:db:auposinstd" (where "xxx" is the current region and "yyy" is the current AWS Account ID)

	* In the "fixtags" element you must change the value of the key "check.dbarn":

		**[original value]** "dbarn": "arn:aws:rds:CHANGE_ME:CHANGE_ME:db:auposinstd"

		**[new value]** "dbarn": "arn:aws:rds:xxx:yyy:db:auposinstd" (where "xxx" is the current region and "yyy" is the current AWS Account ID)

## Test the solution

### Test 1: Cloning an existing Aurora cluster using Aurora Fast-cloning

1. Execute the Step Functions state machine "state-machine-awssol" using the Python script **launch_refresh.py** (from the package **awssoldb-orchestrator-launch.zip**)

	* $ cd awssoldb
	* $ cd awssoldb-orchestrator-launch
	* $ python3 launch_refresh.py auposinstd app2 <state-machine-arn> <region>

1. Monitor the status of the cloning operation using the Step Functions dashboard and the RDS dashboard

1. At the end of the cloning operation you will find a new entry in the DynamoDB table "dbalignment-awssol" and you will get an e-mail with a confirm about the cloning just completed

### Test 2: Cloning an existing RDS database instance through a Point-In-Time-Restore

1. Execute the Step Functions state machine "state-machine-awssol" using the Python script **launch_refresh.py** (from the package **awssoldb-orchestrator-launch.zip**)

	* $ cd awssoldb
	* $ cd awssoldb-orchestrator-launch
	* $ python3 launch_refresh.py mysqlinstd app1 <state-machine-arn> <region>

1. Monitor the status of the cloning operation using the Step Functions dashboard and the RDS dashboard

1. At the end of the cloning operation you will find a new entry in the DynamoDB table "dbalignment-awssol" and you will get an e-mail with a confirm about the cloning just completed

### Test 2b: Cloning an existing RDS database instance through a Point-In-Time-Restore (with Secrets Manager support)

This test requires a new Secrets Manager secret that will be associated with the new restored RDS database instance. In this way, during the refresh, once the master user password of the new database is changed, the secret will be updated accordingly:

1. Create a Secrets Manager secret in the same region where you deployed the solution. The secret must have the following characteristics:

	* Secret type: **Credentials for RDS database**

	* User name: **admin*

	* Password: the one you find in the refresh file **db-app1-mysqlinstd.json** (from the package **awssoldb-orchestrator-launch.zip**)

	* Database: **mysqlinstd**

	* Secret name: choose the name you prefer (suggestion: /development/app1/mysqlinstd)

	* Rotation: **disabled**

In the previous step by launching our solution you created from scratch a new database ("mysqlinstd"). Before trigger a new refresh you must change the related refresh file in order to let solution stop and delete the existing cloned database:

1. Modify the refresh file **db-app1-mysqlinstd.json** (from the package **awssoldb-orchestrator-launch.zip**):
	* In the "stopdb" element you must change the value for both the keys "torun" and "check.torun":

		**[original value]** "torun": "false"

		**[new value]** "torun": "true"

	* In the "delete" element you must change the value for both the keys "torun" and "check.torun":

		**[original value]** "torun": "false"

		**[new value]** "torun": "true"

	* In the "changemasterpwd" element you must change the value of the key "secret":

		**[original value]** "secret": "false"

		**[new value]** "secret": "true"

	* In the "changemasterpwd" element you must add the following new key-value pair (you can put it after the "secret" one):

		**[original value]** "secretname": "CHANGE_ME"

		**[new value]** "secretname": "xxx" (where "xxx" is the name of the secret previously created)

1. Execute the Step Functions state machine "state-machine-awssol" using the Python script **launch_refresh.py** (from the package **awssoldb-orchestrator-launch.zip**)

	* $ cd awssoldb-orchestrator-launch
	* $ python3 launch_refresh.py mysqlinstd app1 arn:aws:states:<region>:<account_id>:stateMachine:state-machine-awssol <region>

1. Monitor the status of the cloning operation using the Step Functions dashboard and the RDS dashboard

1. At the end of the cloning operation you will find a new entry in the DynamoDB table "dbalignment-awssol" and you will get an e-mail with a confirm about the cloning just completed

### Test 3c: Cloning an existing RDS database instance through a Point-In-Time-Restore (with Secrets Manager support and post-refresh scripts)

In this test we will execute the SQL scripts uploaded on S3 in the section Pre-requirements. These scripts represent post-refresh scripts executed against the new restored RDS database instance. The scripts will be executed by the Lambda function "awssoldb-RunScriptsMySQL". 

1. The function needs to be modified to include the PyMySQL library (see https://pypi.org/project/PyMySQL/), the open source Python library used to connect to MySQL databases in this test:

	* Download the PyMySQL library (source https://pypi.org/project/PyMySQL/#files) and save the .tar.gz file under ./awssoldb-orchestrator-pkg-cloudformation/functions/extra-awssoldb-RunScriptsMySQL/
	* Extract the PyMySQL-x.xx.x.tar.gz file
	* Move the directory "pymysql" and the file "LICENSE.txt" under ./awssoldb-orchestrator-pkg-cloudformation/functions/extra-awssoldb-RunScriptsMySQL/ 
	* $ cd ./awssoldb-orchestrator-pkg-cloudformation/functions/extra-awssoldb-RunScriptsMySQL/
	* $ zip -r awssoldb-RunScriptsMySQL.zip awssoldb-RunScriptsMySQL.py LICENSE.txt pymysql
	* $ aws lambda update-function-code --function-name "awssoldb-RunScriptsMySQL" --zip-file fileb://awssoldb-RunScriptsMySQL.zip --region us-east-1

1. Modify the refresh file **db-app1-mysqlinstd.json** (from the package **awssoldb-orchestrator-launch.zip**):
	* In the "runscripts" element you must add the following new key-value pair (you can put it after the "secret" one):

		**[original value]** "secretname": "CHANGE_ME"

		**[new value]** "secretname": "xxx" (where "xxx" is the name of the secret previously created)

	* In the "runscripts" element you must change the value for both the keys "torun" and "check.torun":

		**[original value]** "torun": "false"

		**[new value]** "torun": "true"

1. Execute the Step Functions state machine "state-machine-awssol" using the Python script **launch_refresh.py** (from the package **awssoldb-orchestrator-launch.zip**)

	* $ cd awssoldb-orchestrator-launch
	* $ python3 launch_refresh.py mysqlinstd app1 arn:aws:states:<region>:<account_id>:stateMachine:state-machine-awssol <region>

1. Monitor the status of the cloning operation using the Step Functions dashboard and the RDS dashboard

1. At the end of the cloning operation you will find a new entry in the DynamoDB table "dbalignment-awssol" and you will get an e-mail with a confirm about the cloning just completed

1. As the last step, verify the results of the SQL scripts execution:

	* Using your own SQL Client, connect to the database "mysqlinstd" and run the content of the SQL script **s3://awssol-bucket/sqlscripts/rdsmysql/mysqlinstd/final_check.sql**

	* Take a look at the results of the last three SELECT statements you will get and compare them with the previous ones. A table useless in the development environment was truncated and sensitive information such as customer's e-mails and addresses were masked

## Clean up

To delete everything, you need to:

1. Delete the new Aurora cluster created during "Test 1"

1. Delete the new RDS database instance created during "Test 2"

1. Delete the parent CloudFormation stack, the one associated with the **awssoldb_global.template** template. The deletion of all the stacks should take around 10 minutes

1. Delete the two S3 buckets created as pre-requirements

1. Delete the SNS topic created as pre-requirements

1. Delete the EC2 key pair created as pre-requirements


## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.
