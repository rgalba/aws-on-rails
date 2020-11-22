# AWS on Rails

## AWS Cloud9 setup

In order to setup a Cloud9 env, follows the steps to create it using aws cli.
Requirements:

- AWS account
- AWS CLI properly configured

1. Creates the environment:

```
aws cloudformation create-stack --stack-name rails-cloud9 --template-body file://automation/cloud9.yml
```

> If you want to check the stack progress, you can watch it with: `watch aws cloudformation describe-stack-events --stack-name rails-cloud9`

2. Get the URL to access the environment:

```
aws cloudformation describe-stacks --stack-name rails-cloud9 --query "Stacks[0].Outputs[0].OutputValue" --output text | pbcopy
```

## Setting up

(Optional) Use `guru` profile:
> export AWS_PROFILE=guru

1 - Create key pair

```sh
aws ec2 create-key-pair --key-name rails-key-pair --query 'KeyMaterial' --output text > rails-key-pair.pem
chmod 400 rails-key-pair.pem
```

2 - Create security group

```sh
SG_ID=$(aws ec2 create-security-group --group-name rails-ecs-sg --description 'security group for ECS' --output text)
```

3 - Authorize SSH and HTTP for public access

```sh
aws ec2 authorize-security-group-ingress --group-id $SG_ID  --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $SG_ID  --protocol tcp --port 80 --cidr 0.0.0.0/0
```

4 - Create role //TODO fix

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

```sh
aws iam create-role --role-name rails-ecs-role  --assume-role-policy-document file://rails-ecs-role.json
```

## Create an ECS cluster

```sh
$ aws ecs create-cluster --cluster-name rails-cluster
{
    "cluster": {
        "clusterArn": "arn:aws:ecs:us-east-1:643006215472:cluster/deep-dive",
        "clusterName": "deep-dive",
        "status": "ACTIVE",
        "registeredContainerInstancesCount": 0,
        "runningTasksCount": 0,
        "pendingTasksCount": 0,
        "activeServicesCount": 0,
        "statistics": [],
        "tags": [],
        "settings": [
            {
                "name": "containerInsights",
                "value": "disabled"
            }
        ],
        "capacityProviders": [],
        "defaultCapacityProviderStrategy": []
    }
}
```

```sh
# aws ecs list-clusters
# aws ecs describe-clusters --clusters rails-cluster
# brew install ossp-uuid
# sudo yum install uuid
# us-east-1
$ aws ec2 run-instances --image-id ami-2b3b6041 --count 1 --instance-type t2.micro --iam-instance-profile Name=railsInstanceRole --key-name rails-key-pair --security-group-ids $SG_ID --user-data file://user-data
$ INSTANCE_ID=$(aws ec2 describe-instances --query 'Reservations[1].Instances[].InstanceId' --output text)
```

## List cluster

```sh
$ CONTAINER_INSTANCE_ID=$(aws ecs list-container-instances --cluster rails-cluster --query 'containerInstanceArns[0]' --output text)
$ aws ecs describe-container-instances --cluster rails-cluster --container-instances $CONTAINER_INSTANCE_ID
```

## Registar a Task definition

```
aws ecs register-task-definition --cli-input-json file://web-task-definition.json
```
