# EKS Cluster with eksctl
 
## Description
 
In this project we create an Amazon EKS cluster and managed node group with a single `eksctl` command, instead of repeating the manual console setup (cluster, IAM roles, VPC, node group) from our previous project.
 
`eksctl` is a CLI tool that automates these tasks through CloudFormation, and we can also use it to manage the cluster after it is created.
 
## Technologies Used
 
- Kubernetes
- AWS EKS
- eksctl
- AWS CLI
- kubectl
- Linux / macOS terminal
## Prerequisites
 
- An AWS account. We deleted the cluster from our previous project, so we start clean.
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and [kubectl](https://kubernetes.io/docs/tasks/tools/) installed.
- An AWS user with enough permissions to create EKS, EC2, VPC, IAM and CloudFormation resources. We use the admin user we created in AWS.
## Steps
 
### 1. Install eksctl
 
For macOS:
 
```sh
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl
```
 
### 2. Configure AWS credentials
 
`eksctl` needs our AWS credentials to create the resources. We check which credentials, config file and region are active:
 
```sh
aws configure list
```
 
### 3. Create the cluster
 
We run `eksctl create cluster` and override the defaults we care about:
 
```sh
eksctl create cluster \
  --name demo-cluster \
  --version 1.36 \
  --region eu-central-2 \
  --nodegroup-name demo-nodes \
  --node-type t3.small \
  --nodes 2 --nodes-min 1 --nodes-max 3 \
  --with-oidc
```