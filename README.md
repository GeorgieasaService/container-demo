# Creating a containerized website on ECS Fargate
## Instructions

## 1. Create a cluster
We need to create a cluster to run our container in. This is a logical grouping of EC2 instances. We will use the Fargate launch type, which means that we don't need to manage the EC2 instances ourselves.
We can create a cluster using the following command:
```
aws ecs create-cluster --cluster-name ecs-demo-cluster
```

Let's confirm that the cluster has been created by running the following command:
```
aws ecs list-clusters
```

## 2. Register a task definition
```
aws ecs register-task-definition --cli-input-json file://task-definition.json

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
Once you've decided on which vpc and subnet to use in this project copy their respective vpc and subnet ID's to your notes.

Next, enter this command to create your security group, replace the vpc-id with the one you found above:
```
aws ec2 create-security-group \
--group-name ecs-demo-security-group --description "security group for ecs demo" \
--vpc-id vpc-your-vpc-id \
--tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=my-security-group}]'
```

Use this command to find the security group ID:
```
aws ec2 describe-security-groups \
--filters "Name=group-name,Values=ecs-demo-security-group" \
--query "SecurityGroups[0].GroupId" \
--output text
```

Use this command to find your ip address:
```
curl ifconfig.me
```

Use this command to allow your ip address to access the security group on port 80:
```
aws ec2 authorize-security-group-ingress \
--group-id your-security-group-id \
--protocol tcp --port 80 \
--cidr your-ip/32
```

Next, we'll create the service:
Replace the subnet and security group id's with the ones you found above.
```
aws ecs create-service \
--cluster ecs-demo-cluster \
--service-name ecs-demo-service \
--task-definition ecs-demo-task:1 \
--desired-count=1 \
--launch-type "FARGATE" \
--network-configuration "awsvpcConfiguration={subnets=[subnet-your-subnet-id], securityGroups=[sg-your-sg-id], assignPublicIp=ENABLED}"
```

Use these commands to confirm that the service and task have been created:
``` 
aws ecs list-services --cluster ecs-demo-cluster
```

## 4. Test the service
To test the running task, we need to find the public IP address of the task.

To do so we need the ARN of the task. Use this command to list all running tasks and retrieve the ARN:
```
aws ecs list-tasks --cluster ecs-demo-cluster
```

Use this command to retrieve details of the running task. We want to note down it's eni-id
```
aws ecs describe-tasks \
--cluster ecs-demo-cluster \
--tasks "your-task-id" \
--query "tasks[0].attachments[0].details[?name=='networkInterfaceId'].value" --output text
```
Copy this eni-id to your notes because you'll need it in the next command.

Using your cluster name and the above task id arn, run the following command to describe the task:
```
aws ecs describe-tasks \
--cluster ecs-demo-cluster \
--tasks your-task-id-arn
```
Copy the eni-id to your notes.

To find the public IP address of the task, run the following command:
```
aws ec2 describe-network-interfaces \
--network-interface-id your-eni-id
```

Copy the public IP address and paste it into your browser. You should see the following page:
![NGINX webpage](https://github.com/GeorgieasaService/container-demo/assets/67550608/9f9ecbcd-020c-4457-bc7e-2c0fcb578c3f)


## 5. Clean up
To clean up, we need to delete the service and the cluster. We can do this by running the following commands:
```
aws ecs delete-service \
--cluster ecs-demo-cluster \
--service ecs-demo-service \
--force
```

To confirm that the service has been deleted, run the following command:
```
aws ecs list-services --cluster ecs-demo-cluster
```
