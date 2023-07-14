# Creating EBS snapshots 

### Creating EBS snapshots of all volumes of an EC2 instance and sending e-mail notification via sns topic as a disaster recovery plan.

As part of a disaster recovery plan, we need to take frequent backup or our EBS volumes attached to our EC2 instances.

In this case we are using a Python script to perform this operation.

We first create an EC2 instance and assign necessary tags. The tags will be used as a filter for our python script to find and create snapshots of all the EBS volumes attached to the instances.

### Launch EC2 Instance
![1](https://imgur.com/jJ2ZtCc.jpg)

<br/>

![2](https://imgur.com/lJJZpZS.jpg)

<br/>

### Add tags
![3](https://imgur.com/Di2fCr5.jpg)

<br/>

### Creating SNS topic
Create a SNS topic which will send e-mail notification 
![4](https://imgur.com/RhD0Jjv.jpg)

<br/>

![5](https://imgur.com/3RfPVLS.jpg)

<br/>

![6](https://imgur.com/mJjupaE.jpg)

<br/>

### Create Subscription
This will send your an e-mail which you need to confirm
![7](https://imgur.com/6aychmw.jpg)

<br/>

### Confirm Subscription
![8](https://imgur.com/SoJ413a.jpg)

<br/>

### Python code for our project


```python
import boto3

ec2 = boto3.resource('ec2')
sns_client = boto3.client('sns')

snapshot_ids = []

for instance in ec2.instances.filter(Filters=[
        {
            'Name': 'tag:Backup',
            'Values': [
                'Yes',
            ]
        },
    ]):
    for vol in instance.volumes.all():
        snapshot = vol.create_snapshot(Description='Created by boto3')
        snapshot_ids.append(snapshot.snapshot_id)

sns_client.publish(
    TopicArn='arn:aws:sns:us-east-1:112233445566:snapshots', # insert ARN
    Subject='EBS Snapshots',
    Message=str(snapshot_ids)
)
            
```


