# 🚀 Multi-Region 3-Tier Application Deployment Using AWS CloudFormation & CodePipeline

## This repository contains the complete implementation of a fully automated multi-region CI/CD pipeline that deploys a 3-tier architecture (VPC → RDS → EC2 Web App) to two AWS regions:

Development (us-east-1)

Production (us-west-2)

The entire deployment is driven by AWS CodePipeline, CodeBuild, CloudFormation, S3, and EC2, with automated validation, artifact packaging, and multi-region orchestration.

### 🏗️ Architecture Overview
✔ 3-Tier Architecture

## The system follows a standard 3-tier model:

1. Presentation Tier (Web Tier)

EC2 instance running Apache

Serves an HTML website from /var/www/html/index.html

HTML file comes from the CI/CD build artifact (S3)

2. Application Tier

EC2 UserData handles:

Package installation

Apache configuration

Artifact download from S3

Deployment of website content

3. Data Tier

Amazon RDS MySQL in private subnets

Security group restricts access to only the app server security group
# STEPS FOLLOWED
## 1) I had all my yaml templates prepared
- here is a breakdown of how the look.
## 🧩 CloudFormation Templates

- The stack is broken into 3 templates:

### 📁 templates/vpc.yaml

Creates:

VPC

Public/Private subnets

Route tables

Internet Gateway

NAT Gateway

Exports VPC resources (used by DB + App stacks)

### 📁 templates/db.yaml

Creates:

RDS MySQL DB

DB subnet group

DB security group

Exports DB endpoint

### 📁 templates/app.yaml

Creates:

Security group

EC2 instance

Apache web server

Deployment of index.html from S3 artifacts bucket

Tags instance with ForceUpdate to force rebuilds

UserData performs:

yum update -y
yum install -y httpd awscli
systemctl start httpd
aws s3 cp s3://<ArtifactBucket>/app/index.html /var/www/html/index.html

### 🔁 CI/CD Pipeline (CodePipeline)

The pipeline.yaml template defines a multi-region pipeline with these stages:

1. Source Stage

Pulls from GitHub:

2. Build Stage (CodeBuild)

The buildspec.yml performs:

artifact S3 bucket.

3. DeployDev Stage (us-east-1)

Deploys in this order:

App stack

DB stack

4. Manual Approval

You must approve before production.

5. DeployProd Stage (us-west-2)

### 🧪 Using CFN-Lint,i validated all CloudFormation templates locally using:
```bash
cfn-lint vpc.yaml
cfn-lint db.yaml
cfn-lint app.yaml
cfn-lint pipeline.yaml
```
---

This helped detect:

Missing parameters

Indentation problems

Incorrect intrinsic functions

Wrong ImportValue references

🧹 Cleaning Old Pipeline

You removed existing pipeline conflicts
<img width="1920" height="926" alt="Screenshot (812)" src="https://github.com/user-attachments/assets/3e9c2e5f-4db0-4bc6-b6f8-d5f3833602e9" />
<img width="1920" height="960" alt="Screenshot (813)" src="https://github.com/user-attachments/assets/09ff7335-8d81-4c90-bf79-4a05f032396b" />

### Created buckets for both region.
<img width="1920" height="954" alt="Screenshot (985)" src="https://github.com/user-attachments/assets/f637f2ca-0a04-407e-b8e4-ec311a5c19a8" />
### github connection
<img width="1920" height="833" alt="Screenshot (760)" src="https://github.com/user-attachments/assets/d4f4f52e-27ca-4b8f-811e-20ba5a21be26" />


### Deploy stacks 
- Sample commands
```bash
aws cloudformation create-stack \
  --stack-name cicd-pipeline-stack \
  --template-body file://pipeline.yaml \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
```
<img width="1920" height="938" alt="Screenshot (965)" src="https://github.com/user-attachments/assets/3d08d5c6-0f16-4bb1-908c-11d67c69b257" />
<img width="1920" height="930" alt="Screenshot (964)" src="https://github.com/user-attachments/assets/83d6980a-a80a-4ac8-8f7b-d1be4bb253d4" />
<img width="1920" height="950" alt="Screenshot (925)" src="https://github.com/user-attachments/assets/56af1868-a006-4473-a842-9b43ae8c6edf" />

 ### check the created stacks
- stacks in us-east-1 (dev enviroment)
<img width="1920" height="785" alt="Screenshot (1044)" src="https://github.com/user-attachments/assets/f3db345c-f575-4793-b4b8-07e789ca6ea0" />
- stacks in us-west-2 (production)
<img width="1920" height="806" alt="Screenshot (1045)" src="https://github.com/user-attachments/assets/7e42bc1b-f802-434e-ac45-1aef3fd9c81f" />
- Resources created:
- Costom VPC stack in both dev and production region
<img width="1920" height="900" alt="Screenshot (1065)" src="https://github.com/user-attachments/assets/8c717d36-bf85-4056-81a7-1dc375bc2bd8" />
<img width="1920" height="931" alt="Screenshot (1064)" src="https://github.com/user-attachments/assets/1b485d41-89a9-4164-b3af-49eb2ca75d6d" />
- Database stacks for both region
<img width="1920" height="902" alt="Screenshot (1042)" src="https://github.com/user-attachments/assets/da3f587d-e2ca-4df4-8529-45eb5ba07c76" />
<img width="1920" height="861" alt="Screenshot (1043)" src="https://github.com/user-attachments/assets/2ecac4d3-498e-4690-accf-fb1a53111cb8" />
- App stacks for Dev and Prod
<img width="1920" height="821" alt="Screenshot (1040)" src="https://github.com/user-attachments/assets/1119ddac-8f99-4500-9d4a-f7b73062bd4c" />
<img width="1920" height="819" alt="Screenshot (1041)" src="https://github.com/user-attachments/assets/a050b45d-1e45-41e9-96f1-9304f871d0d3" />
# Tested connectivity between the serve and the database
<img width="1920" height="944" alt="Screenshot (876)" src="https://github.com/user-attachments/assets/fffd0d0a-7744-4637-88fb-3b1712f8e50f" />
<img width="1920" height="927" alt="Screenshot (878)" src="https://github.com/user-attachments/assets/0f737314-ed4c-440a-8dff-298e394e97a6" />
### Pipeline overview
- successfully deploy at the dev enviroment
<img width="1920" height="930" alt="Screenshot (1006)" src="https://github.com/user-attachments/assets/1f01e46f-482c-488e-bb6e-f0b164e44604" />
- Check if everthing was working good and approve manually
<img width="1920" height="774" alt="Screenshot (1037)" src="https://github.com/user-attachments/assets/17e73e8e-f625-4f84-8088-5c65fe6deb5a" />
<img width="1920" height="873" alt="Screenshot (1038)" src="https://github.com/user-attachments/assets/154285c5-ef2a-490f-a47e-36723b07b256" />
- Verifying if everthing works
<img width="1920" height="926" alt="Screenshot (1060)" src="https://github.com/user-attachments/assets/3248a175-b8a6-4110-a0f5-27e7891dd2fc" />


## 🟢 End-to-End Flow

Push change to GitHub

Pipeline pulls the code

CodeBuild validates & packages templates and index.html

Build artifacts uploaded to S3

Dev stack deploys automatically

Manual approval

Prod stack deploys to us-west-2

EC2 retrieves index.html from S3 and deploys website

🎉 Result

You now have a fully automated, multi-region, production-ready CI/CD pipeline for a complete 3-tier AWS application.

This setup is suitable for:

DevOps projects

Cloud engineering portfolios

Real production workloads

Multi-region disaster recovery demonstrations


