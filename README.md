# terraform-aws-eks-efs-storageclass

If your eks cluster was set up by eks module, You can refer to below configuration.

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

anything else

```hcl
data "aws_eks_cluster" "dev-nimtechnology-engines" {
  name = var.cluster_id
}


module "eks-efs-csi-driver" {
  source  = "mrnim94/eks-efs-csi-driver/aws"
  version = "1.0.6"

  aws_region = var.aws_region
  environment = var.environment
  business_divsion = var.business_divsion

  eks_cluster_certificate_authority_data = data.aws_eks_cluster.dev-nimtechnology-engines.certificate_authority[0].data
  eks_cluster_endpoint = data.aws_eks_cluster.dev-nimtechnology-engines.endpoint
  eks_cluster_id = var.cluster_id
  aws_iam_openid_connect_provider_arn = "arn:aws:iam::${element(split(":", "${data.aws_eks_cluster.dev-nimtechnology-engines.arn}"), 4)}:oidc-provider/${element(split("//", "${data.aws_eks_cluster.dev-nimtechnology-engines.identity[0].oidc[0].issuer}"), 1)}"
}
```

You can apply my example to testing:

```yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
  labels:
    name: pvc1
spec:
  storageClassName: efs-sc-nimtechnology
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi

---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: myapp
spec:
  selector:
    matchLabels:
      name: myapp
  template:
    metadata:
      name: myapp
      labels:
        name: myapp
    spec:
      volumes:
      # Khai báo VL sử dụng PVC
      - name: myvolume
        persistentVolumeClaim:
          claimName: pvc1
      containers:
      - name: myapp
        image: busybox
        resources:
          limits:
            memory: "50Mi"
            cpu: "500m"
        command:
          - sleep
          - "600"
        volumeMounts:
        - mountPath: "/data"
          name: myvolume
```