# PROJECT 24 - BUILDING ELASTIC KUBERNETES SERVICE (EKS) WITH TERRAFORM

Since project 21 and 22, you have had some fragmented experience around kubernetes bootstraping and deployment of containerised applications. This project seeks to solidify your skills by focusing more on real world set up.

- You will use Terraform to create a Kubernetes EKS cluster and dynamically add scalable worker nodes
- You will deploy multiple applications using HELM
- You will experience more kubernetes objects and how to use them with Helm. Such as Dynamic provisioning of volumes to make pods stateful
- You will improve upon your CI/CD skills with Jenkins

In Project 21, you created a k8s cluster from Ground-Up. That was quite painful, but very necessary to help you master kubernetes. Going forward, you will not have to do that. Even in the real world, you will hardly ever have to do that. given that cloud providers such as AWS have managed services for kubernetes, they have done all the hard work, and with a few API calls to AWS, you can have a production grade cluster ready to go in minutes. Therefore, in this project, you begin by focusing on EKS, and how to get it up and running using Terraform. Before moving on to other things.

Building EKS with Terraform
At this point you already have some Terraform experience. So, you have some work to do. But, you can get started with the steps below. If you have terraform code from Project 16, simply update the code and include EKS starting from number 6 below. Otherwise, follow the steps from number 1

Note: Use Terraform version v1.0.2 and kubectl version v1.23.6

Open up a new directory on your laptop, and name it eks
Use AWS CLI to create an S3 bucket
Create a file – backend.tf Task for you, ensure the backend is configured for remote state in S3
terraform {
}
Create a file – network.tf and provision Elastic IP for Nat Gateway, VPC, Private and public subnets.

# reserve Elastic IP to be used in our NAT gateway

resource "aws_eip" "nat_gw_elastic_ip" {
vpc = true

tags = {
Name = "${var.cluster_name}-nat-eip"
iac_environment = var.iac_environment_tag
}
}
Create VPC using the official AWS module
module "vpc" {
source = "terraform-aws-modules/vpc/aws"

name = "${var.name_prefix}-vpc"
cidr = var.main_network_block
azs = data.aws_availability_zones.available_azs.names

private_subnets = [

# this loop will create a one-line list as ["10.0.0.0/20", "10.0.16.0/20", "10.0.32.0/20", ...]

# with a length depending on how many Zones are available

for zone_id in data.aws_availability_zones.available_azs.zone_ids :
cidrsubnet(var.main_network_block, var.subnet_prefix_extension, tonumber(substr(zone_id, length(zone_id) - 1, 1)) - 1)
]

public_subnets = [

# this loop will create a one-line list as ["10.0.128.0/20", "10.0.144.0/20", "10.0.160.0/20", ...]

# with a length depending on how many Zones are available

# there is a zone Offset variable, to make sure no collisions are present with private subnet blocks

for zone_id in data.aws_availability_zones.available_azs.zone_ids :
cidrsubnet(var.main_network_block, var.subnet_prefix_extension, tonumber(substr(zone_id, length(zone_id) - 1, 1)) + var.zone_offset - 1)
]

# Enable single NAT Gateway to save some money

# WARNING: this could create a single point of failure, since we are creating a NAT Gateway in one AZ only

# feel free to change these options if you need to ensure full Availability without the need of running 'terraform apply'

# reference: https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/2.44.0#nat-gateway-scenarios

enable_nat_gateway = true
single_nat_gateway = true
one_nat_gateway_per_az = false
enable_dns_hostnames = true
reuse_nat_ips = true
external_nat_ip_ids = [aws_eip.nat_gw_elastic_ip.id]

# Add VPC/Subnet tags required by EKS

tags = {
"kubernetes.io/cluster/${var.cluster_name}" = "shared"
iac_environment                             = var.iac_environment_tag
}
public_subnet_tags = {
"kubernetes.io/cluster/${var.cluster_name}" = "shared"
"kubernetes.io/role/elb" = "1"
iac_environment = var.iac_environment_tag
}
private_subnet_tags = {
"kubernetes.io/cluster/${var.cluster_name}" = "shared"
"kubernetes.io/role/internal-elb" = "1"
iac_environment = var.iac_environment_tag
}
}
Note: The tags added to the subnets is very important. The Kubernetes Cloud Controller Manager (cloud-controller-manager) and AWS Load Balancer Controller (aws-load-balancer-controller) needs to identify the cluster’s. To do that, it querries the cluster’s subnets by using the tags as a filter.

For public and private subnets that use load balancer resources: each subnet must be tagged

Key: kubernetes.io/cluster/cluster-name
Value: shared
For private subnets that use internal load balancer resources: each subnet must be tagged

Key: kubernetes.io/role/internal-elb
Value: 1
For public subnets that use internal load balancer resources: each subnet must be tagged

Key: kubernetes.io/role/elb
Value: 1
Create a file – variables.tf

# create some variables

variable "cluster_name" {
type = string
description = "EKS cluster name."
}
variable "iac_environment_tag" {
type = string
description = "AWS tag to indicate environment name of each infrastructure object."
}
variable "name_prefix" {
type = string
description = "Prefix to be used on each infrastructure object Name created in AWS."
}
variable "main_network_block" {
type = string
description = "Base CIDR block to be used in our VPC."
}
variable "subnet_prefix_extension" {
type = number
description = "CIDR block bits extension to calculate CIDR blocks of each subnetwork."
}
variable "zone_offset" {
type = number
description = "CIDR block bits extension offset to calculate Public subnets, avoiding collisions with Private subnets."
}
Create a file – data.tf – This will pull the available AZs for use.

# get all available AZs in our region

data "aws_availability_zones" "available_azs" {
state = "available"
}
data "aws_caller_identity" "current" {} # used for accesing Account ID and ARN
