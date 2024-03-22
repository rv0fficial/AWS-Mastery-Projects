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
3. In the Instances section, create an EC2 instance preferred settings.
4. Next, navigate to the 'Elastic Block Store' section and select 'Volumes'. You will notice that a default volume has already been created for us.
5. Next, click on 'Snapshots,' and then click the 'Create Snapshot' button. It will prompt you with a page that looks like this.
6. In Volume ID section choose your default Volume ID created when we create instance.
7. Next, click 'Next,' provide a name for your Snapshot, and then scroll down and click 'Create Snapshot'.
