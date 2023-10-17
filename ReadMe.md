# README

# DevOps Assignment: Host Docker ROR Application with Nginx in AWS using IaC

## Introduction

This assignment aims to demonstrate the process of deploying a Dockerized Ruby on Rails application with Nginx in AWS with Loadbalancer and RDS using IaC.

### Guideline for the assignment submission

1. Fork this repo into your GitHub account. Setup the build process from the source code. Build process should generate a Docker image and upload it to AWS ECR.
2. Create a new folder named "infrastructure" in the root of the project and push your IaC code under this folder.
3. Prepare a Terraform/CloudFormation/CDK script to provision the scalable infrastructure in AWS ECS/EKS using this ECR image. You need to use ELB to distribute the traffic between servers. All the resources should be hosted in private subnet except load balancer.
4. The web application will integrate with database and S3. So you may need to create a RDS instance (Postgres) and S3 bucket and use them as ENV variable in ECS. Required ENV names will be mentioned in Github repo’s README. Application should integrate with S3 using IAM role authentication, not AccessKey and SecretKey. Application should integrate with RDS using database credentials (Host, DB name, Username and Password).
5. It should also contain a ReadMe file with cleardetails about how to use and create the Infrastructure with this IaC code.
6. Prepare an architecture diagram, deployment steps, and any other relevant information in the same folder.
7. Share a GitHub repository to Github account “Mallowtechdev” and send an email to HR team (hr@mallow-tech.com) about the completion along with Github repository link and branch details.


### Iac Structure
...
AWSTemplateFormatVersion: '2010-09-09'
Description: AWS CloudFormation Template for complete application setup

Parameters:
  VpcCidrBlock:
    Type: String
    Default: '10.0.0.0/16'

  PublicSubnetCidrBlock1:
    Type: String
    Default: '10.0.1.0/24'

  PublicSubnetCidrBlock2:
    Type: String
    Default: '10.0.2.0/24'

  PrivateSubnetCidrBlock1:
    Type: String
    Default: '10.0.3.0/24'

  PrivateSubnetCidrBlock2:
    Type: String
    Default: '10.0.4.0/24
	
  ECRRepositoryName:
    Type: String	
    Default: 'ror-app'

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCidrBlock

  InternetGateway:
    Type: AWS::EC2::InternetGateway

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Ref PublicSubnetCidrBlock1
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true

  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Ref PublicSubnetCidrBlock2
      AvailabilityZone: !Select [1, !GetAZs '']
      MapPublicIpOnLaunch: true

  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Ref PrivateSubnetCidrBlock1
      AvailabilityZone: !Select [0, !GetAZs '']

  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Ref PrivateSubnetCidrBlock2
      AvailabilityZone: !Select [1, !GetAZs '']

  MyECRRepository:
    Type: AWS::ECR::Repository
    Properties:
      RepositoryName: !Ref ECRRepositoryName

  RDSInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      AllocatedStorage: '20'
      StorageType: 'gp2'
      DBInstanceClass: 'db.t2.micro'
      Engine: 'postgres'
      EngineVersion: '13.3'
      MasterUsername: 'postgres'
      MasterUserPassword: 'dfHwroJ1rt43'
      DBName: 'postgres'
      MultiAZ: false
      VPCSecurityGroups:
        - !GetAtt SecurityGroup.GroupId
      DBSubnetGroupName: !Ref DBSubnetGroup

  DBSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: 'Subnet group for RDS'
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2

  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'Security Group for ECS tasks'
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '5432'
          ToPort: '5432'
          SourceSecurityGroupId: !GetAtt SecurityGroup.GroupId

  ECSCluster:
    Type: AWS::ECS::Cluster

  LoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Subnets:
        - !Ref PublicSubnet1
        - !Ref PublicSubnet2
      SecurityGroups:
        - !GetAtt SecurityGroup.GroupId

Outputs:
  VpcId:
    Description: 'VPC ID'
    Value: !Ref VPC

  RDSInstanceEndpoint:
    Description: 'RDS instance endpoint'
    Value: !GetAtt RDSInstance.Endpoint.Address

  ECSClusterName:
    Description: 'ECS Cluster Name'
    Value: !Ref ECSCluster

  LoadBalancerDNSName:
    Description: 'DNS name of the Load Balancer'
    Value: !GetAtt LoadBalancer.DNSName
...


### Prerequisites

1. AWS Account with appropriate permissions.
2. RDS Postgres 13.3 Database ( update the credentials in the below mentioned Environment variables)
3. LoadBalancer  ( update the loadbalancer endpoint in the environment variable)
4. Docker installed in your local machine.
5. Your preferred IaC tool ( Terraform, CDK, CloudFormation)
6. Other local tools if required.

### Version details:

* Ruby version - `3.2.2`
* Rails version - `7.0.5`
* Database - `Postgresql - 13.3`

### Docker conatiner details

* Rails container running in port 3000
* Nginx container running in port 80
* You can use the container alias name "rails_app" to for the nginx to send request to rails container


### Docker Folder Structure

    ...
    ├── docker
    │   ├── app
    │   │   ├── Dockerfile         # Rails container dockerfile
    │   │   └── entrypoint.sh      # Rails container entrypoint
    │   └── nginx
    │       ├── default.conf       # Nginx config file
    │       └── Dockerfile         # Nginx container dockerfile
    │                   
    ├── docker-compose.yml         # docker-compose file
    ...

### Environment variable for Ruby container

```env
RDS_DB_NAME=rails
RDS_USERNAME=postgres
RDS_PASSWORD=dfHwroJ1rt43
RDS_HOSTNAME=postgres
RDS_PORT=5432
S3_BUCKET_NAME=ror-app1
S3_REGION_NAME=ap-south-1
LB_ENDPOINT="ror-app-1887812992.ap-south-1.elb.amazonaws.com"
```

### Environment variable for Nginx container

```env
nil
```

### Note
This README is a guideline and should be adjusted based on your specific setup and requirements.
