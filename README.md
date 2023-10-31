# Creating a containerized website on ECS fargate
## Instructions

## 1. Create a cluster
We need to create a cluster to run our container in. This is a logical grouping of EC2 instances. We will use the Fargate launch type, which means that we don't need to manage the EC2 instances ourselves.
We can create a cluster using the following command:
```
aws ecs create-cluster --cluster-name your-cluster-name
```
Let's confirm that the cluster has been created by running the following command:
```
aws ecs list-clusters
```

## 2. Register a task definition
```
aws ecs register-task-definition --cli-input-json file://your-task-definition.json
```
We can confirm that the task definition has been registered by running the following command:
```
aws ecs list-task-definitions
```

## 3. Configure the service
You'll have to find a vpc and public subnet to use in your account. You can do this by running the following commands:
```
aws ec2 describe-vpcs

aws ec2 describe-subnets
```
Copy their respective vpc and subnet ID's to your notes

Next, enter this command to create your security group, replace the vpc-id with the one you found above:
```
aws ec2 create-security-group \
--group-name my-security-group --description "my-security-group" \
--vpc-id vpc-your-vpc-id \
--tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=my-security-group}]'
```

Use this command to find the security group ID:
```
aws ec2 describe-security-groups \
--filters "Name=group-name,Values=george-ecs-test" \
--query "SecurityGroups[0].GroupId" \
--output text
```
Use this command to find your ip address:
```
ip addr show eth0 | grep inet | awk '{print $2}' | cut -d '/' -f 1
```

Use this command to allow your ip address to access the security group on port 80:
```
aws ec2 authorize-security-group-ingress --group-id sg-your-security-group-id --protocol tcp --port 80 --cidr your-ip/32
```

Next, we'll create the service:
```
aws ecs create-service \
--cluster your-cluster-name \
--service-name your-service-name \
--task-definition http-server:1 \
--desired-count=1 \
--launch-type "FARGATE" \
--network-configuration "awsvpcConfiguration={subnets=[subnet-your-subnet-id], securityGroups=[sg-your-sg-id], assignPublicIp=ENABLED}"
```
Replace the subnet and security group id's with the ones you found above.

Use these commands to confirm that the service and task have been created:
``` 
aws ecs list-services --cluster your-cluster-name
```

## 4. Test the service
To test the running task, we need to find it's public IP address. We can do this by running the following command:
```
aws ecs describe-tasks --cluster your-cluster-name --tasks your-task-id --query "tasks[0].attachments[0].details[?name=='networkInterfaceId'].value" --output text
```

To list running tasks enter:
```
aws ecs list-tasks --cluster your-cluster-name
```
Copy this ARN to your notes because you'll need it in the next command.

Using your cluster name and the above task id arn, run the following command to describe the task:
```
aws ecs describe-tasks \
--cluster your-cluster-name \
--tasks your-task-id-arn
```
Copy the eni-id to your notes.

To find the public IP address of the task, run the following command:
```
aws ec2 describe-network-interfaces \
--network-interface-id your-eni-id
```
Copy the public IP address and paste it into your browser. You should see the following page:

![NGINX webpage](image.png)

## 5. Clean up
To clean up, we need to delete the service and the cluster. We can do this by running the following commands:
```
aws ecs delete-service \
--cluster your-cluster-name \
--service your-service-name \
--force
```
To confirm that the service has been deleted, run the following command:
```
aws ecs list-services --cluster your-cluster-name
```
