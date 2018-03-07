# Scheduling AWS EC2 instances for shutdown and startup
Running AWS EC2 instances around the clock can be cost-draining, so these two scripts have been used with CloudWatch to schedule a timed
startup and shutdown of all EC2 instances assigned to a particular project.

The scripts run in AWS's own version of Python and are shown below:

* StartEC2Instances.py
```
import boto3
# Enter the region your instances are in. Include only the region without specifying Availability Zone; e.g.; 'us-east-1'
region = 'eu-west-2'

def lambda_handler(event, context):

	def get_tag_value(tags, key):
		for tag in tags:
			if tag['Key'] == key:
				return tag['Value']
		else:
			raise KeyError

	ec2 = boto3.client('ec2', region_name=region)
	instances = []
	ignored_instances = []
	for reservation in ec2.describe_instances()['Reservations']:
		for instance in reservation['Instances']:
			instance_id = instance['InstanceId']
			try:
				tags = instance['Tags']
				value = get_tag_value(tags, 'project')
				if value == 'my-project':
					instances.append(instance_id)
			except KeyError:
				pass
			if instance_id not in instances:
				ignored_instances.append(instance_id)

	if instances:
		ec2.start_instances(InstanceIds=instances)
	print 'ignored instances: ' + str(ignored_instances)
	print 'started your instances: ' + str(instances)
  ```
  * StopEC2Instances.py
  ```
  import boto3
# Enter the region your instances are in. Include only the region without specifying Availability Zone; e.g.; 'us-east-1'
region = 'eu-west-2'

def lambda_handler(event, context):

	def get_tag_value(tags, key):
		for tag in tags:
			if tag['Key'] == key:
				return tag['Value']
		else:
			raise KeyError

	ec2 = boto3.client('ec2', region_name=region)
	instances = []
	ignored_instances = []
	for reservation in ec2.describe_instances()['Reservations']:
		for instance in reservation['Instances']:
			instance_id = instance['InstanceId']
			try:
				tags = instance['Tags']
				value = get_tag_value(tags, 'project')
				if value == 'my-project':
					instances.append(instance_id)
			except KeyError:
				pass
			if instance_id not in instances:
				ignored_instances.append(instance_id)

	if instances:
		ec2.stop_instances(InstanceIds=instances)
	print 'ignored instances: ' + str(ignored_instances)
	print 'stopped your instances: ' + str(instances)
  ```
  To enable these, create two Lambda functions within AWS, pasting the above scripts into the function window and selecting Python 2.7
  as the script language and giving each a meaningful name (ie: StartEC2, StopEC2).
  
  Next, go into CloudWatch and create two rules - one for each function. Assign the newly created function to their respective rules
  and set a schedule for each one (eg: Start rule can be set for 8am, Monday - Friday and Stop rule can be set to run 6pm, Monday -
  Friday).
  
  In the above scripts, the startup or shutdown directive applies to all EC2 instances which have a tag called 'project' with a
  corresponding value of 'my-project'. A table called 'instances' is populated with all the EC2 instances which have a tag called 
  'project' with the 'my-project' value and the directive 'ec2.stop_instances(InstanceIds=instances)' instructs CloudWatch to shut
  down or start up each instance in the table.
