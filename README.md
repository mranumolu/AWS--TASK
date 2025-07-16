# AWS--TASK

⸻

Step 1: Create a VPC with 3 Subnets

1.1 Go to VPC Dashboard
	•	Open AWS Console.
	•	Go to Services > VPC.

1.2 Create VPC
	•	Click Create VPC.
	•	Name: backend-vpc
	•	IPv4 CIDR block: e.g., 10.0.0.0/16
	•	Leave other options default, click Create.

1.3 Create Subnets
	•	Under Subnets, click Create subnet.
	•	VPC: Select backend-vpc
	•	Availability Zones:
	•	Subnet 1:
	•	Name: backend-1a
	•	AZ: Select us-east-1a
	•	CIDR: 10.0.1.0/24
	•	Subnet 2:
	•	Name: backend-1b
	•	AZ: Select us-east-1b
	•	CIDR: 10.0.2.0/24
	•	Subnet 3:
	•	Name: backend-1c
	•	AZ: Select us-east-1c
	•	CIDR: 10.0.3.0/24
	•	Click Create subnet after adding all three.

⸻

Step 2: Create Lambda Function

2.1 Go to Lambda Dashboard
	•	Go to Services > Lambda.

2.2 Create Function
	•	Click Create function > Author from scratch.
	•	Function name: PrintRandomMessage
	•	Runtime: Python 3.12 (or latest)
	•	Permissions: Create a new role with basic Lambda permissions (default is fine).

2.3 Configure VPC for Lambda
	•	In the Lambda function page, click Configuration > VPC.
	•	Click Edit.
	•	Select your VPC (backend-vpc).
	•	Select Subnets:
	•	Choose only backend-1b and backend-1c.
	•	Security Group:
	•	Use the default or create a new one if needed (no special inbound rules needed for this basic use case).
	•	Click Save.

2.4 Add Python Code
	•	In Code tab, replace handler with:
 
"""
import random

def lambda_handler(event, context):
    messages = [
        "Hello from Lambda!",
        "Random number: " + str(random.randint(1, 100)),
        "Your S3 event triggered this Lambda.",
        "Everything is working perfectly!",
    ]
    print(random.choice(messages))
    return {"statusCode": 200}
"""

	•	Click Deploy.

⸻

Step 3: Create S3 Bucket

3.1 Go to S3 Dashboard
	•	Go to Services > S3.

3.2 Create Bucket
	•	Click Create bucket.
	•	Bucket name: Choose a unique name (e.g., my-random-lambda-trigger-bucket)
	•	Region: Match the region of your VPC/Lambda (e.g., us-east-1)
	•	Leave other options default (you can uncheck “Block all public access” for testing, but NOT recommended for production).
	•	Click Create bucket.

⸻

Step 4: Configure S3 Event Notification (Trigger)

4.1 Add Trigger to Lambda
	•	Go back to your Lambda function in AWS Console.
	•	Click Configuration > Triggers.
	•	Click Add trigger.
	•	Select source: S3
	•	Bucket: Select your bucket (my-random-lambda-trigger-bucket)
	•	Event type: All object create events
	•	Prefix/Suffix: Leave blank for all uploads
	•	Enable trigger: Checked
	•	Click Add.

4.2 Lambda Permissions
	•	AWS will automatically update the Lambda execution role to allow S3 to invoke this function.
	•	If not, you may need to add the necessary permissions to your Lambda’s IAM role:
	•	Go to IAM > Roles, find your Lambda role.
	•	Attach the AWSLambdaVPCAccessExecutionRole and AmazonS3ReadOnlyAccess (if you want to access file content).

⸻

Step 5: Test the Setup
	1.	Go to your S3 bucket.
	2.	Click Upload and upload any test file.
	3.	Go to CloudWatch Logs > Log groups > /aws/lambda/PrintRandomMessage (or your function name).
	4.	Open the latest log stream and check that your Lambda printed one of the random messages.

⸻

