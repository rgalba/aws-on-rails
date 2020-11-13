# AWS on Rails

## Pre-requisites

Use `guru` profile:
> export AWS_PROFILE=guru

## Setting up

1 - Create key pair

```sh
aws ec2 create-key-pair --key-name rails-key-pair --query 'KeyMaterial' --output text > rails-key-pair.pem
chmod 400 rails-key-pair.pem
```

2 - Create security group

```sh
SG_ID=$(aws ec2 create-security-group --group-name rails-ecs-sg --description 'security group for ECS')
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
aws ecs list-clusters
aws ecs describe-clusters --clusters rails-cluster
# brew install ossp-uuid
# sudo yum install uuid
BUCKET_NAME=$(uuid)-ecs-rails-cluster
aws s3api create-bucket --bucket $BUCKET_NAME
# us-east-1
$ aws ec2 run-instances --image-id ami-2b3b6041 --count 1 --instance-type t2.micro --iam-instance-profile Name=railsInstanceRole --key-name rails-key-pair --security-group-ids $SG_ID --user-data file://user-data
```

> instance id: 

## List cluster

```sh
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-2b3b6041",
            "InstanceId": "i-0b0bdb30a37c1012d",
            "InstanceType": "t2.micro",
            "KeyName": "rails-key-pair",
            "LaunchTime": "2020-11-13T02:04:16.000Z",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-1e",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-31-56-139.ec2.internal",
            "PrivateIpAddress": "172.31.56.139",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-0fe3f644d8699d4f7",
            "VpcId": "vpc-094f410fc5a1b4384",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "05d30991-dacd-47ee-90ba-1d2b884a372b",
            "EbsOptimized": false,
            "Hypervisor": "xen",
            "IamInstanceProfile": {
                "Arn": "arn:aws:iam::315628488150:instance-profile/railsInstanceRole",
                "Id": "AIPAUS7HMQXLG5FB4IMCU"
            },
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2020-11-13T02:04:16.000Z",
                        "AttachmentId": "eni-attach-0bc5024e5d688ef08",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "rails-ecs-sg",
                            "GroupId": "sg-05d26cefb6ee5e74b"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "06:c1:fa:6c:9d:91",
                    "NetworkInterfaceId": "eni-00065d61506d8c9fa",
                    "OwnerId": "315628488150",
                    "PrivateDnsName": "ip-172-31-56-139.ec2.internal",
                    "PrivateIpAddress": "172.31.56.139",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-31-56-139.ec2.internal",
                            "PrivateIpAddress": "172.31.56.139"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-0fe3f644d8699d4f7",
                    "VpcId": "vpc-094f410fc5a1b4384",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "rails-ecs-sg",
                    "GroupId": "sg-05d26cefb6ee5e74b"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1,
                "HttpEndpoint": "enabled"
            },
            "EnclaveOptions": {
                "Enabled": false
            }
        }
    ],
    "OwnerId": "315628488150",
    "ReservationId": "r-065dce0f09217306c"
}
```

## List cluster

```sh
$ aws ecs list-container-instances --cluster rails-cluster
```

Ideia de Skill: Meus documentos. Um cofre onde a Alexa saberá numeros de documento como CPF, RG, Titulo de eleitor por meio de senha.