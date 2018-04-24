# Scheduling AWS Resources for shutdown and startup
Running AWS resources around the clock can be cost-draining, so these scripts have been setup to schedule two cron jobs to schedule the startup and shutdown of resources based on tags.

The scripts are written in Python 3 using the AWS SDK boto3 and can be found in management/scripts

The project consists of:
  - aws_resource_finders.py - The script with the logic in for finding resources by a tag and startng/stopping them.
  - start_resources_example.py - An example script for use in a Start Resources Lambda function.
  - stop_resources_example.py - An example script for use in a Start Resources Lambda function.

Recommended Use:

The scripts make use of AWS Tags which can be put on most resources. The current scripts make use of a tag with name 'Scheduled' and the value of 'True'. If people are on holiday for example they may wish to set their Scheduled flag to 'False' so that it will not turn on whilst they are away. You can of course change the value to a tag key-pair that suits your project.
  
To enable these, create two Lambda functions within AWS one should have the aws_resource_finders.py and the start_resources.py the other function should have the aws_resource_finders.py and the stop_resources.py. You should modify the examples to enable the resources you need. For example If you have no RDS resources then there is no point checking for them. The language is Python 3, and you will need to increase the timeout of the function as the default of 3 is not long enough usually. The handlers would be start_resources.lambda_handler and stop_resources.lambda_handler as appropriate. Name the Lambda functions as appropriate e.g. Start-Resources and Stop-Resources.
  
Next, go into CloudWatch and create two rules - one for each function. Assign the newly created function to their respective rules and set a schedule for each one (eg: Start rule can be set for 8am, Monday - Friday and Stop rule can be set to run 6pm, Monday - Friday). Example cron expressions are:
  - 0 8 ? * MON-FRI *    8 am every monday-friday
  - 0 18 ? * MON-FRI *   6 pm every monday-friday
  
You are encouraged to extend the project by adding other resources into the aws_resource_finders.py that may be of use.