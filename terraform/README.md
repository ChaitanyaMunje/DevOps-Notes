# Terraform in One Shot

---

## Terraform vs Ansible

- **Terraform**: Used for provisioning infrastructure (for example, creating servers, databases, networks, etc.).
- **Ansible**: Used to configure or manage infrastructure, typically after it has been provisioned by Terraform.

---

## Terraform vs CloudFormation

- **Terraform**: Can provision infrastructure on any cloud provider—AWS, Azure, GCP, and more.
- **CloudFormation**: AWS’s own infrastructure provisioning tool, similar to Terraform, but only for AWS resources.

---

## Installation of Terraform

- **Using Homebrew on macOS:**
  ```sh
  brew tap hashicorp/tap
  brew install hashicorp/tap/terraform
  ```
- **Official documentation:**  
  [Install Terraform](https://developer.hashicorp.com/terraform/install)

---

## Coding in Terraform

Terraform code is organized into blocks that contain parameters and arguments.
```
<Block> "<Type>" "<Name>" {
  argument1 = "value"
  argument2 = "value"
}
```
### Common Blocks:
- `resource`: To define a cloud resource (e.g., AWS EC2 instance, S3 bucket)
- `output`: To display values after execution
- `variable`: To define input parameters

### Example

```hcl
resource "local_file" "my_file" {
    filename = "example.txt"
    content  = "This is an example file created by Terraform."
}
```
- `resource`: Block type
- `"local_file"`: Provider/type of resource
- `"my_file"`: Name/label for this resource
- `filename`, `content`: Arguments (user-defined properties)

**Arguments**: Values provided by the user before running a plan.  
**Attributes**: Values generated during/after execution, visible after a plan/apply.

---

## Terraform Workflow

1. **Navigate** to the directory with your `.tf` files.
2. Run:
    - `terraform init` &nbsp;&nbsp;&nbsp; _(Initializes Terraform and downloads providers)_
    - `terraform validate` &nbsp;&nbsp;&nbsp; _(Validates configuration)_
    - `terraform plan` &nbsp;&nbsp;&nbsp; _(Shows planned changes - no actual changes yet)_
    - `terraform apply` &nbsp;&nbsp;&nbsp; _(Creates/manages real infrastructure)_
    - `terraform destroy` &nbsp;&nbsp;&nbsp; _(Removes managed infrastructure)_

---

## Providers in Terraform

Providers let Terraform manage resources on a specific platform. They are defined in a `terraform` block.

**Example: Adding the AWS and Azure provider**
```hcl
terraform { 
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "6.28.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "4.58.0"
    }
  }
}
```
- See provider documentation at the [Terraform Registry](https://registry.terraform.io/providers/hashicorp/azurerm/latest).

**After editing providers, re-run:**  
`terraform init`

**AWS CLI installation:**  
[Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

---

## Example: Creating an S3 Bucket

**1. `terraform.tf`:**
```hcl
terraform { 
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "6.28.0"
    }
  }
}
```

**2. `provider.tf`:**
```hcl
provider "aws" {
  region = "ap-south-1"
}
```

**3. `s3.tf`:**
```hcl
resource "aws_s3_bucket" "mybucket" {
  bucket = "test-terraform-bucket-ch"
}
```

**Commands:**
```sh
terraform init
terraform plan
terraform apply
# To Delete:
terraform destroy
```

---

## Example: Creating an EC2 Instance

### Step-by-Step

**Step 1: `terraform.tf`**
```hcl
terraform { 
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "6.28.0"
    }
  }
}
```

**Step 2: `provider.tf`**
```hcl
provider "aws" {
  region = "ap-south-1"
}
```

**Step 3: `variables.tf`**
```hcl
variable "ec2_instance_type" {
  description = "Type of AWS EC2 instance"
  type        = string
  default     = "t2.micro"
}
variable "ec2_root_storage_size" {
  description = "Size of the root storage volume in GB"
  type        = number
  default     = 8
}
variable "ec2_ami_id" {
  description = "AMI ID for the EC2 instance"
  type        = string
  default     = "ami-019715e0d74f695be" # Ubuntu 20.04 in us-east-1
}
```

**Step 4: `outputs.tf`**
```hcl
output "ec2_public_ip" {
  value = aws_instance.my_instance.public_ip
}
output "ec2_public_dns" {
  value = aws_instance.my_instance.public_dns
}
output "ec2_private_ip" {
  value = aws_instance.my_instance.private_ip
}
```

**Step 5: `ec2.tf`**

- Generate a key pair for SSH access (`ssh-keygen`).

```hcl
# Key Pair
resource "aws_key_pair" "my_key" {
  key_name   = "terra-key-ec2"
  public_key = file("terra-key-ec2.pub")
}

# VPC (default)
resource "aws_default_vpc" "default" {}

# Security Group
resource "aws_security_group" "my_security_group" {
  name        = "automate_sg"
  vpc_id      = aws_default_vpc.default.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow SSH from anywhere"
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow HTTP from anywhere"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow all outbound traffic"
  }

  tags = {
    Name = "automate_sg"
  }
}

# EC2 Instance
resource "aws_instance" "my_instance" {
  ami               = var.ec2_ami_id
  instance_type     = var.ec2_instance_type
  key_name          = aws_key_pair.my_key.key_name
  security_groups   = [aws_security_group.my_security_group.name]
  user_data         = file("install_nginx.sh")

  root_block_device {
    volume_size = var.ec2_root_storage_size
    volume_type = "gp3"
  }

  tags = {
    Name = "Instance Name"
  }
}
```

---

**Execution:**
```sh
terraform init
terraform plan
terraform apply
```
To list resources: `terraform state list`  
To destroy a single resource:  
`terraform destroy -target <resource_name>`

---

## Creating Multiple EC2 Instances

Use `for_each` to generate several similar resources:

```hcl
resource "aws_instance" "my_instance" {
  for_each = tomap({
    "tf_instance_one" = "t2.micro"
    "tf_instance_two" = "t2.micro"
  })

  depends_on      = [ aws_security_group.my_security_group, aws_key_pair.my_key ]
  ami             = var.ec2_ami_id
  instance_type   = each.value
  key_name        = aws_key_pair.my_key.key_name
  security_groups = [aws_security_group.my_security_group.name]
  user_data       = file("install_nginx.sh")

  root_block_device {
    volume_size = var.ec2_root_storage_size
    volume_type = "gp3"
  }

  tags = {
    Name = each.key
  }
}
```
- Number of elements in `for_each` map determines the number of instances.
- `each.key`: Name, `each.value`: Instance type.

---

## State Management in Terraform

- **State file** (`terraform.tfstate`) tracks current cloud resources as known by Terraform.
- **Refresh state:** `terraform refresh`  
  (Also automatically refreshed on `terraform apply`)
- **List state resources:** `terraform state list`

### Remote State Management

- Store `.tfstate` remotely (e.g., in an S3 bucket) so that multiple team members can collaborate.
- Use **DynamoDB** for locking to ensure only one person can update the state at a time.

---

## References

- [Terraform Installation Guide](https://developer.hashicorp.com/terraform/install)
- [Terraform AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest)
- [Terraform Azure Provider Docs](https://registry.terraform.io/providers/hashicorp/azurerm/latest)
- [AWS CLI Installation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

---
