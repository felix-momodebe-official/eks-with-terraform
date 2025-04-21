# Terraform AWS EKS Cluster Deployment

This repository contains Terraform code to provision an Amazon EKS (Elastic Kubernetes Service) cluster on AWS with the necessary infrastructure components.

## Overview

The Terraform configuration creates and manages the following AWS resources:

- VPC with 2 public subnets across different availability zones
- Internet Gateway for public internet access
- Route tables and associations
- Security groups for the EKS cluster and node groups
- EKS cluster with appropriate IAM roles and policies
- EKS node group with remote SSH access capability

## Prerequisites

Before using this Terraform code, ensure you have:

1. [Terraform](https://www.terraform.io/downloads.html) installed (version 0.12 or later)
2. AWS CLI installed and configured with appropriate credentials
3. An AWS EC2 key pair for SSH access to worker nodes (default: "project4")
4. `kubectl` installed to interact with the EKS cluster after deployment

## Architecture

The infrastructure creates:

- **VPC**: A dedicated VPC with CIDR block `10.0.0.0/16`
- **Subnets**: 2 public subnets in us-west-1a and us-west-1b availability zones
- **Internet Gateway**: Attached to the VPC for public internet access
- **Route Tables**: Configured to route traffic through the Internet Gateway
- **Security Groups**:
  - Cluster Security Group: Controls access to the EKS control plane
  - Node Security Group: Controls access to the worker nodes, allowing all inbound traffic
- **EKS Cluster**: Running the Kubernetes control plane
- **EKS Node Group**: 3 worker nodes of type t2.large with SSH access

## Resource Details

### Networking Resources
- **VPC**: Named "devopsola-vpc" with CIDR block 10.0.0.0/16
- **Subnets**: Two public subnets with auto-assigned public IPs
- **Internet Gateway**: Allows communication between instances in the VPC and the internet
- **Route Table**: Routes all external traffic through the Internet Gateway

### Security Resources
- **Cluster Security Group**: Allows all outbound traffic
- **Node Security Group**: Allows all inbound and outbound traffic

### EKS Resources
- **EKS Cluster**: Named "devopsola-cluster" with appropriate IAM roles
- **Node Group**: 3 t2.large instances running as worker nodes
- **IAM Roles**:
  - Cluster Role: With AmazonEKSClusterPolicy permissions
  - Node Group Role: With necessary policies for worker nodes

## Provider Configuration

This project uses the AWS provider with the following configuration:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

## Variables

| Name | Description | Type | Default |
|------|-------------|------|---------|
| ssh_key_name | The name of the SSH key pair to use for instances | string | "project4" |

## Outputs

| Name | Description |
|------|-------------|
| cluster_id | The ID of the EKS cluster |
| node_group_id | The ID of the EKS node group |
| vpc_id | The ID of the VPC |
| subnet_ids | The IDs of the subnets |

## Usage

### Deployment Steps

1. Clone this repository to your local machine:
```bash
git clone <repository-url>
cd <repository-directory>
```

2. Initialize Terraform:
```bash
terraform init
```

3. Review the execution plan:
```bash
terraform plan
```

4. Apply the configuration:
```bash
terraform apply
```

5. When prompted, confirm the action by typing `yes`.

### Connecting to the Cluster

After successful deployment, configure kubectl to use your new EKS cluster:

```bash
aws eks update-kubeconfig --region us-east-1 --name devopsola-cluster
```

Verify the connection:

```bash
kubectl get nodes
```

## Region Note

This configuration deploys the EKS cluster in the `us-east-1` region, while the subnet configuration references `us-west-1` availability zones. You may need to adjust the subnet availability zones to match the provider region or change the provider region to `us-west-1`.

## Security Considerations

The current configuration permits all inbound traffic to worker nodes and all outbound traffic from both worker nodes and the control plane. In a production environment, consider restricting these security group rules to only necessary traffic.

## Cleanup

To destroy all resources created by Terraform:

```bash
terraform destroy
```

When prompted, confirm the action by typing `yes`.

## License

[MIT License](LICENSE)
