# AWS Cloud Cost Optimization - Identifying Stale Resources

#### Problem :
It's common for developers to create EC2 instances with attached volumes and create snapshots for backup purposes. 
However, when these instances are terminated, developers may forget to delete the associated snapshots, leading to ongoing costs for unused resources. 

#### Solution :
Creating a Lambda function to automatically manage snapshots and EC2 instances is a smart cost-saving measure. 
This will effectively prevent unnecessary storage costs by periodically scanning for EBS snapshots not associated with active EC2 instances and deleting them.

#### Description:
The Lambda function fetches all EBS snapshots owned by the same account ('self') and also retrieves a list of active EC2 instances (running and stopped). 
For each snapshot, it checks if the associated volume (if exists) is not associated with any active instance. If it finds a stale snapshot, it deletes it, e
ffectively optimizing storage costs.

**Note:** Indeed, it's important to note that there are many similar issues, such as forgetting to release resources like Elastic IP after terminating EC2 instances. 
To address this issue, additional Lambda functions have to be implemented. In this case only foucs on Stale EBS Snapshots.

## Steps

### Step 1:
1. Log into your AWS Console.
2. Navigate to the EC2 Console.
3. In the Instances section, create an EC2 instance with preferred settings.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/92bb3114-1c97-4c54-9380-8115f0ce7a3f)
4. Next, navigate to the `Elastic Block Store` section and select `Volumes`. 
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/e7226679-a74b-48d9-bb2e-c6fc32cf2d8f)
  You will notice that a default volume has already been created with the EC2 instance.
5. Next, click on `Snapshots`, and then click the `Create Snapshot` button. It will direct you to the following page.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/f6d425bc-77b4-45c5-a608-86762c5b3beb)
  In the Volume ID section choose your `default Volume ID` of the created EC2 instance.
6. After that, provide a name for your Snapshot using `tags`, and then scroll down and click `Create Snapshot`.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/b48b2374-b716-4a7c-9ee1-28dc2b3c26b1)

### Step 2 :
1. After creating a Snapshot, navigate to the `Lambda` Console.
2. You will see some options in the user interface, such as `Create Function`.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/c0ab7014-2b48-4605-a804-078477d8b6e4)
  Click on `Create Function`.
3. Select `Author from Scratch`, then enter the `Function name`, and choose the latest `Python` version.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/5828f751-4457-4179-bb2a-30cffc59d063)
  We will go with the default permission and update the permission later. Scroll down and click `create Function`.
4. After creating the function, scroll down, and you will see something like the image below.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/5075afc3-04bf-4107-8fd8-c1b8746de4c6)
   Click on the `Code` section.
5. Next, clear the existing code and replace it with the `identify_stale_snapshots.py` code.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/a1b680e5-2997-43a7-b88b-50b0d93fd219)
6. Click `Deploy` to save your changes, and then click `Test`. It will direct you to the following page.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/ca233ef0-112f-4dfb-a1cd-896ddee7bf54)
  Please configure the settings as displayed above and then scroll down. Next, click on `Save`.
7. Once you've created the event, proceed to the IAM Console(Identity and Access Management) and then navigate policies section to create a new policy (Press `Create policy`).
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/872d88ee-76f3-4081-b98d-5da4be4601db)
8. Select the service as `EC2`
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/13bdd5ed-6fc9-4763-86f8-1b3b78622a5d)
9. In the 'Actions' section, grant permissions for the following actions: `DescribeInstances`, `DescribeVolumes`, `DescribeSnapshots`, `DeleteSnapshot`.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/67f26895-120f-4454-bfe9-48021aa82d20)
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/07d35eac-d21b-4a13-a1f0-d9607441b536)
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/a4fa49d2-523e-43f8-bd1a-ba9017bfe51a)
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/de0a71d7-b932-4154-9c30-2622c63e557f)
  Then scroll down and press `Next`
10. Give a `Policy name`, scroll down and click `Create policy`
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/a0343682-d014-4dae-8698-ce4ef9bae838)
11. After Creating the Policies , Navigate to the Lambda Sections.
12. Next, go to the page of the Lambda function you've created. In the "Permissions" section, click on the role name.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/d2c7c26e-bdb2-40fc-9018-87a809485d87)
13. Click on 'Add Permissions' and then select 'Attach Policy.'
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/e401000a-7e00-4ee4-9cc7-ef5357783b38)
14. Choose the correct policy you created.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/943e7d67-a1e4-43b8-baf0-d58cfbb28c62)
  Then scroll down and click `Add Permissions`.
15. After that, you can go to the Lambda function page and run the code using `Test` button; it will display some outputs as shown below.
![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/6dccee49-57c8-4dc5-8f16-f46c960ccb1a)

**Note:** If you come across an [error](https://stackoverflow.com/questions/62948910/aws-lambda-errormessage-task-timed-out-after-3-00-seconds) like the one mentioned in the attached link, then you need to follow the steps below.
![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/09ebbb49-3db9-46a6-b77b-ffc042bc987e)
![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/3e95e695-af55-4101-a597-893f45b1b9aa)

### Step 3 :
1. You can terminate the EC2 instance to test our Lambda function.
2. Navigate to the EC2 console and then terminate the EC2 instance.
3. Return to the Lambda console to test the code; go to the Lambda Function page.
4. Under the Code section, click `Test` code, it will display an output as follows.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/c58df32f-ab0c-4379-bde8-a7073ac96029)
5. As expected, our Lambda function deleted the snapshot because it was associated with a volume that couldn't be found.

### Additional notes :
It's worth emphasizing that while the lambda function operates on events, in the given scenario, you initiated the lambda function manually. To automate this process, CloudWatch can be employed to trigger the Lambda function at specified intervals such as hourly, daily, or even minutely. However, this automation may escalate costs due to increased Lambda execution time when triggered automatically. Nonetheless, manually initiating the function is preferable as it enables us to trigger it precisely when necessary.

## CloudWatch or EventBridge Implementation :
### Steps :
1. Navigate to CloudWatch Console and go to events `Rules`.
2. Next, on the following page, configure the schedule pattern as follows:
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/2a7787c8-aeff-4dda-99c0-19b5f66744c3)
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/6dab37c4-0c8b-4972-9725-25830b78828a)
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/04b76e9b-af26-4c05-85f1-9f729490c78c)
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/f92afc54-fe14-4182-b7ad-1775d3c5f555)
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/11a8c61e-26ab-4e01-8172-e607962b1800)
  Scroll Down and then Click `Next`.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/e4a35b9d-b6d9-4392-bd79-a6fbe2da424f)
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/9e4055cb-9722-4be7-a625-2111e878428b)
  Scroll Down and then Click `Next`.
3. On the next page, choose `None` for the `Action after Schedule` option.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/19ad6faf-4429-4763-8a00-c51ed0abe091)
  keep the rest as it is.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/1fc74aef-d9e9-4716-8cdb-e4af4ddb2096)
  Scroll Down and then Click `Next`.
  ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/14bc4a0b-af5e-433a-986a-7cf11c632fc4)
  keep the rest as it is, Scroll Down and then Click `Next` in above screenshot.
4. You have successfully created the scheduler, which will trigger the Lambda function every hour.
5. However, please note that this setup will incur some costs since the function is triggered continuously every hour. Alternatively, we can configure it to run on specific days and times as needed.
