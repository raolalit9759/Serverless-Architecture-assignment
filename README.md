# Serverless-Architecture-assignment
Serverless-Architecture-assignment
Automated Instance Management Using AWS Lambda and Boto3:
1. Set Up EC2 Instances:
1. Create EC2 Instances:
Create First Instance (Auto-Stop): Add Tags: Under Value, type ‘Auto-Stop’. Create second Instance (Auto-Start): Add Tags: Under Value, type ‘Auto-Start’.

2. Set Up IAM Role for Lambda:
Create IAM Role: select AmazonEC2FullAccess policy. Give the role a name ‘LambdaEC2Role’ and click Create Role.

3. Create Lambda Function:
Create the Function Function Name: EC2AutoStartStop. Runtime: Select Python 3.x. Permissions: Choose the IAM role LambdaEC2Role.

4. Write the Lambda Code:
In the Lambda function code editor, paste the following Python code using Boto3:

import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')

    # Find instances with tag 'Action' = 'Auto-Stop'
    stop_instances = ec2.describe_instances(
        Filters=[
            {'Name': 'tag:Action', 'Values': ['Auto-Stop']}
        ]
    )

    # Find instances with tag 'Action' = 'Auto-Start'
    start_instances = ec2.describe_instances(
         Filters=[
             {'Name': 'tag:Action', 'Values': ['Auto-Start']}
         ]
    )

    stop_instance_ids = [instance['InstanceId'] 
                        for reservation in stop_instances['Reservations']
                        for instance in reservation['Instances']]

    start_instance_ids = [instance['InstanceId'] 
                         for reservation in start_instances['Reservations']
                         for instance in reservation['Instances']]

    # Stop 'Auto-Stop' instances
    if stop_instance_ids:
       ec2.stop_instances(InstanceIds=stop_instance_ids)
       print(f'Stopped instances: {stop_instance_ids}')

    # Start 'Auto-Start' instances
    if start_instance_ids:
       ec2.start_instances(InstanceIds=start_instance_ids)
       print(f'Started instances: {start_instance_ids}')

     return {
         'statusCode': 200,
          'body': 'Lambda executed successfully!'
            }
Click Deploy to save the Lambda function

5. Test the Lambda Function
Manually Invoke the Function Create a new test event Click Test again to run the function. Check EC2 Instances Go back to the EC2 Dashboard. Verify that the instance tagged with Auto-Stop is stopped, and the one tagged with Auto- Start is started.

2. Implement a Log Cleaner for S3:
1. Set Up the IAM Role
Create a New Role: Attach the AmazonS3FullAccess policy for now. Attach the AWSLambdaBasicExecutionRole for logging in CloudWatch. Assign the Role to your Lambda function under the “Permissions” tab.

2. Create a New Lambda Function
Go to AWS Lambda Console Choose "Author from scratch": Name your function S3LogCleaner. Runtime: Choose Python 3.10. Set Permissions: Create a new IAM role with basic Lambda execution permissions. Attach the AmazonS3FullAccess policy for this exercise. In a real-world scenario, use a more restrictive policy.

3. Write the Boto3 Code for Log Cleaner
import boto3
from datetime import datetime, timezone

 # Initialize S3 client
 s3 = boto3.client('s3')

 def lambda_handler(event, context)
 bucket_name = 'abhay-bucket-cleanup'

# Specify the retention period (90 days)
retention_days = 90
retention_period = datetime.now(timezone.utc) - timedelta(days=retention_days)

# List all objects in the specified S3 bucket
try:
    objects = s3.list_objects_v2(Bucket=bucket_name)
    
    if 'Contents' in objects:
        for obj in objects['Contents']:
            obj_key = obj['Key']
            obj_last_modified = obj['LastModified']
            
            # Check if the log file is older than retention period
            if obj_last_modified < retention_period:
                print(f"Deleting {obj_key} last modified on {obj_last_modified}")
                s3.delete_object(Bucket=bucket_name, Key=obj_key)
    else:
        print(f"No objects found in bucket {bucket_name}")

except Exception as e:
    print(f"Error: {str(e)}")

return {
    'statusCode': 200,
    'body': 'Log cleaner executed successfully.'
}
4. Schedule the Function with EventBridge:
To ensure the Lambda function runs weekly, we will schedule it using AWS EventBridge Go to EventBridge: Open the Amazon EventBridge console. Click on Create rule. Configure Rule Details: Name: Name the rule WeeklyS3LogCleanup. Event Source: Choose Event Source as EventBridge Scheduler. Schedule: Choose the schedule expression as rate (7 days) to trigger the function weekly. Target: Add a Target. Select Lambda Function as the target and choose the Lambda function you created.

5. Test the Function
After setting everything up, test the function manually to ensure it works before the weekly schedule kicks in.

Go to Lambda and click Test function.
Verify the Logs: Check the Lambda logs in CloudWatch to see which files were deleted.
Go to the S3 bucket and verify that files older than 90 days have been removed.
3. Automated S3 Bucket Cleanup Using AWS Lambda and Boto3:
1. Set Up S3 Bucket
Log into AWS Console: Navigate to AWS Management Console. Go to the S3 Dashboard: In the search bar, type S3 and click on S3. Click Create Bucket. Create the Bucket: Name of bucket my-bucket-cleanup and choose a region. Leave the default settings for Block Public Access and other configurations. Click Create Bucket. Upload Files to the Bucket

2. Set Up IAM Role for Lambda
In the AWS Console, open IAM Dashboard. click Create Role select AWS Service and choose Lambda. Click Next, then search for and select AmazonS3FullAccess policy. Give the role a name LambdaS3Role and click Create Role.

3. Create Lambda Function
Create the Function Go to the Lambda Dashboard Function Name: S3CleanupFunction. Runtime: Select Python 3.10. Permissions: Choose the IAM role LambdaS3Role. Click Create Function.

4. Write the Lambda Code
Click Deploy to save the Lambda function.

import boto3
 from datetime import datetime, timezone, timedelta

 # Initialize S3 client
s3 = boto3.client('s3')

def lambda_handler(event, context):
bucket_name = 'abhay-bucket-cleanup'

# Specify the retention period (30 days)
retention_days = 30
retention_period = datetime.now(timezone.utc) - timedelta(days=retention_days)

# List all objects in the bucket
objects = s3.list_objects_v2(Bucket=bucket_name)

# Check if any objects are found
if 'Contents' in objects:
    for obj in objects['Contents']:
        obj_key = obj['Key']
        obj_last_modified = obj['LastModified']
        
        # If the object is older than 30 days, delete it
        if obj_last_modified < retention_period:
            print(f"Deleting {obj_key}, last modified on {obj_last_modified}")
            s3.delete_object(Bucket=bucket_name, Key=obj_key)
else:
    print(f"No objects found in bucket {bucket_name}")

return {
    'statusCode': 200,
    'body': 'Cleanup completed.'
}
5. Test the Lambda Function
Manually Invoke the Function In the Lambda function dashboard, click Test. Create a new test event. Click Test again to run the function. Check S3 Bucket Go back to the S3 Dashboard. Verify that files older than 30 days have been deleted, and newer files are still present.

4. Automatic EBS Snapshot and Cleanup Using AWS Lambda and Boto3:
1. Set Up EBS Volume
Go to EC2 Dashboard: In the search bar, type EC2 and click on EC2 Dashboard. On the left side, click Volumes under Elastic Block Store. Identify Volume: Select an existing volume you want to back up, or click Create Volume.

2. Set Up IAM Role for Lambda
Create IAM Role In the AWS Console, search for IAM and open IAM Dashboard. On the left sidebar, click Roles, then click Create Role. Under Trusted Entity Type, select AWS Service and choose Lambda. Click Next, then search for and select AmazonEC2FullAccess policy. Give the role a name LambdaEBSRole, and click Create Role.

3. Create Lambda Function
Create the Function Go to the Lambda Dashboard Fill in the details: Function Name: EBSSnapshotFunction. Runtime: Select Python 3.10. Permissions: Choose the IAM role you created earlier LambdaEBSRole. Click Create Function.

4. Write the Lambda Code
Click Deploy to save the Lambda function.

  import boto3
  import datetime

def lambda_handler(event, context):
ec2 = boto3.client('ec2')

volume_id = 'vol-0451b4186b4b37298'

# Create a snapshot
snapshot = ec2.create_snapshot(VolumeId=volume_id, Description='Automated backup')
snapshot_id = snapshot['SnapshotId']
print(f"Created snapshot: {snapshot_id}")

# Get the current date and set the retention period (30 days)
current_date = datetime.datetime.now(datetime.timezone.utc)
retention_days = 30
retention_date = current_date - datetime.timedelta(days=retention_days)

# Describe and delete snapshots older than 30 days for the specified volume
snapshots = ec2.describe_snapshots(Filters=[{'Name': 'volume-id', 'Values': [volume_id]}])['Snapshots']

for snap in snapshots:
    start_time = snap['StartTime']
    snapshot_id = snap['SnapshotId']
    
    # Compare snapshot creation time with retention date
    if start_time < retention_date:
        ec2.delete_snapshot(SnapshotId=snapshot_id)
        print(f"Deleted snapshot: {snapshot_id}")
    else:
        print(f"Snapshot {snapshot_id} is newer than 30 days, skipping deletion.")

return {
    'statusCode': 200,
    'body': 'Snapshot creation and cleanup process completed!'
}
5. Test the Lambda Function
Manually Invoke the Function In the Lambda function dashboard, click Test. Create a new test event. Click Test again to run the function. Check Snapshots Go to the EC2 Dashboard and select Snapshots under Elastic Block Store. Confirm that a new snapshot was created, and older snapshots were deleted.

6. Set Up Event Source (Bonus)
Go to Amazon CloudWatch in the AWS Console. On the left sidebar, click Rules, then Create Rule. Under Event Source, choose Event Source Type as Schedule: Define your schedule. Under Targets, choose Lambda Function, and select your EBSBackupCleanup Lambda function. Click Create to save the CloudWatch rule.

Conclusion
This assignment provided an in-depth exploration of automating infrastructure management using a serverless architecture with AWS Lambda and Boto3. By automating tasks such as starting/stopping EC2 instances and managing EBS snapshots and backups, we’ve seen how serverless computing can significantly reduce operational overhead, improve cost efficiency, and enhance scalability.

Key takeaways from this assignment include:

Automation and Efficiency: AWS Lambda allows you to automate infrastructure tasks without the need for maintaining a dedicated server, reducing costs and increasing operational efficiency.

Scalability: Serverless solutions are inherently scalable. AWS Lambda dynamically allocates resources, ensuring that functions can handle workloads of any size without manual intervention.

Reduced Management Overhead: Serverless architecture frees up the need for managing servers, patches, and updates, allowing you to focus on business logic rather than infrastructure management.

Cost-effectiveness: With AWS Lambda, you only pay for the compute time you use, making it a cost-effective solution for sporadic or variable workloads.

Contact
For any issues, feel free to reach out:

Name:Lalit Kumar Rao

Email: raolalit9759@gmail.com

GitHub: https://github.com/raolalit9759/
