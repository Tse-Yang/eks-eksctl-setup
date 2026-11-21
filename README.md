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
![eksctl create cluster command](images/01-eksctl-create-command.png)
 
`eksctl` picks three availability zones and creates a public and a private subnet in each one.
 
The command takes around 15 minutes. It deploys the CloudFormation stacks for the cluster and the node group, creates the add-ons, and saves a kubeconfig locally so `kubectl` can talk to the new cluster.
 
![eksctl create cluster output](images/02-eksctl-create-output.png)
 
### 4. Verify the nodes with kubectl
 
```sh
kubectl get nodes
```
 
We see the 2 desired nodes with their private IPs, region and Kubernetes version.
 
![kubectl get nodes](images/03-kubectl-get-nodes.png)
 
### 5. Check the resources in AWS
 
**EKS cluster**: in the AWS console, we open *Amazon EKS > Clusters > demo-cluster*. The status is **Active**.
 
![EKS cluster in the console](images/04-eks-console-cluster.png) <br><br>
 
![EKS nodes](images/05-eks-console-nodes.png)
 
**Workloads**: under *Resources > Pods* we see our cluster running these system components:
 
- `aws-node`: enables pod networking and communication with our AWS account
- `coredns` and `kube-proxy`: core Kubernetes components needed to run scheduled pods and configure services

![EKS workloads](images/06-eks-console-workloads.png)
 
**What `eksctl` created in the background:**
 
- **IAM**: new roles and policies, such as the `AWSServiceRoleForAmazonEKS` service-linked role.
- **VPC**: a dedicated `eksctl-demo-cluster/VPC` with a public and a private subnet in each of the 3 availability zones.
- **CloudFormation**: one stack for the cluster and one for the node group.
### 6. Clean up
 
When we are done, we delete the cluster:
 
```sh
eksctl delete cluster --name demo-cluster --region eu-central-2
```