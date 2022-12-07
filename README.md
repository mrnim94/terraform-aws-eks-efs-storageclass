# terraform-aws-eks-efs-storageclass

```hcl
# Input Variables
# AWS Region
variable "aws_region" {
  description = "Region in which AWS Resources to be created"
  type = string
  default = "us-east-1"  
}

# Terraform Remote State Datasource - Remote Backend AWS S3
data "terraform_remote_state" "eks" {
  backend = "s3"
  config = {
    bucket = "terraform-on-aws-eks-nim"
    key    = "dev/eks-cluster/terraform.tfstate"
    region = var.aws_region
  }
}

module "eks-efs-storageclass" {
  source  = "mrnim94/eks-efs-storageclass/aws"
  version = "1.0.5"

  efs_name = "nimtechnology"

  eks_cluster_certificate_authority_data = data.terraform_remote_state.eks.outputs.cluster_certificate_authority_data
  eks_cluster_endpoint = data.terraform_remote_state.eks.outputs.cluster_endpoint
  eks_cluster_id = data.terraform_remote_state.eks.outputs.cluster_id

  eks_private_subnets = data.terraform_remote_state.eks.outputs.private_subnets
  vpc_cidr_block = data.terraform_remote_state.eks.outputs.vpc_cidr_block
  vpc_id = data.terraform_remote_state.eks.outputs.vpc_id
}
```