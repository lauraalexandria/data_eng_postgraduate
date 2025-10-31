# **Data Engineering Postgraduate**

## Table of Contends

- [🐳 Docker Usage and Key Concepts](#-docker-usage-and-key-concepts)
- [🏗️ Terraform](#-terraform)

# 🐳 **Docker Usage and Key Concepts**

## Container Operations

### Basic Ubuntu Container Execution
To run an Ubuntu terminal environment:
```bash
docker run -it ubuntu bash
```

### Container Management Commands
- `docker ps`: Displays all currently running containers
- `docker run` options:
  - `--name`: Assigns a specific name to the container
  - `-it`: Interactive mode, opening a terminal session for direct interaction
  - `-d`: Detached mode, executing the container in the background
  - `-p`: Port mapping configuration (virtual_port:local_port)

## Dockerfile Fundamentals

The Dockerfile serves as a blueprint for creating Docker images, though it notably lacks a file extension. This design choice reflects its specialized purpose within the Docker ecosystem.

The image creation process requires executing `docker build`, which constructs an image from the Dockerfile specifications. The `FROM PYTHON` directive initiates image creation based on an existing Python image, which subsequently executes the specified script.

The trailing period in build commands (`.`) instructs Docker to locate the Dockerfile in the current directory. During execution, `docker run` first searches for images locally before querying Docker Hub if no local match is found.

## Docker Compose for Multi-Service Management

Docker Compose enables orchestration of multiple services through a single configuration file:

- `-p`: Assigns a project name to the container cluster
- `up`: Creates and starts containers (requires pre-existing or downloadable images)
- `down`: Stops and removes containers
- `-d`: Detached mode operation for background execution

The initial `docker-compose up` execution may require additional time, while subsequent runs benefit from cached layers, resulting in significantly improved performance.

## Disk Management and Cache Usage

As Docker operates, it progressively stores cache data on the hard drive, which can rapidly consume significant storage space. Despite the storage impact, this caching mechanism significantly accelerates subsequent Dockerfile executions, reducing what were previously extended processing times. Verification with `docker system df` to confirm the storage utilized.

# 🏗️ **Terraform**

[Terraform](https://www.terraform.io) is an Infrastructure as Code (**IaC**) management platform. The platform allows you to write code in your preferred programming language to describe your infrastructure in terms of resources, such as servers, networks, and storage volumes. Terraform uses this description to create and manage these resources across multiple cloud providers, such as AWS, Azure, and GCP, among others. One of Terraform's main advantages is that it allows you to manage your infrastructure abstractly, **independent of the specific cloud provider**. This means that, with a single code description, you can manage your infrastructure across multiple cloud providers and even migrate your resources from one provider to another. Furthermore, Terraform allows for code versioning, making change control and rollback easier. And Terraform is free.

## 📁 **Basic Terraform Structure**

### 📄 **Configuration Files**
- **Extension**: `.tf`
- **Main file**: `main.tf` - defines resources and providers
- **Variables**: `variables.tf` - variable declaration
- **Values**: `terraform.tfvars` - variable value assignment
- **Outputs**: `outputs.tf` - displays information after execution

### 🗑️ **Files to Ignore in Git**
```gitignore
# .gitignore for Terraform
.terraform/
*.tfstate
*.tfstate.backup
.terraform.lock.hcl
.DS_Store  # 🍎 macOS folder system file
```

**💡 It's even important to delete the `.DS_Store` file frequently.**

### 🔧 **Fundamental Commands**
```bash
terraform init    # 🏁 Initializes project (creates .terraform/ and .terraform.lock.hcl)
terraform apply   # 🚀 Applies automation
terraform destroy # 💥 Destroys resources (--auto-approve skips confirmation)
```

## 🏗️ **HCL Structure (HashiCorp Configuration Language)**

### 📝 **Main Blocks**
```hcl
# Provider (no longer mandatory in recent versions)
provider "aws" {
  region = "us-east-2"
}

# Resource - defines cloud resources
resource "aws_instance" "web_server" {
  ami           = "ami-*****************"
  instance_type = "t3.micro"
  key_name      = var.key_pair
}

# Variable - declares variables
variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
}
```

## 🎯 **Recommended File Standards**

### 📋 **Ideal Structure**
```
project/
├── main.tf          # 🔧 Main resources
├── variables.tf     # 📊 Variable declaration
├── terraform.tfvars # ⚙️ Variable values
├── outputs.tf       # 📤 Project outputs
└── .gitignore       # 🙈 Ignores .terraform/, .tfstate, .DS_Store
```

## 🔄 **Multiple Instance Creation**

### 👥 **Method 1: Count**
```hcl
variable "instance_count" {
  description = "Number of EC2 instances"
  type        = number
  default     = 3
}

variable "create_instance" {
  description = "Flag to create instances"
  type        = bool
  default     = true
}

resource "aws_instance" "web_server" {
  count = var.create_instance ? var.instance_count : 0  # 🎯 Conditional
  
  ami           = "ami-*****************"
  instance_type = "t3.micro"
  
  tags = {
    Name = "WebServer-${count.index}"  # 🏷️ Unique name for each
  }
}
```

### 🗺️ **Method 2: For Each**
```hcl
variable "web_servers" {
  description = "Web servers configuration"
  type = map(object({
    name = string
  }))
  default = {
    "server1" = { name = "Server1" }
    "server2" = { name = "Server2" }
    "server3" = { name = "Server3" }
  }
}

resource "aws_instance" "web_server" {
  for_each = var.web_servers  # 🔄 Iterates over map
  
  ami           = "ami-*****************"
  instance_type = "t3.micro"
  
  tags = {
    Name = each.value.name  # 🏷️ Uses map name
  }
}
```

## 📤 **Outputs and Modules**

### 📊 **Output Block**
```hcl
output "instance_ids" {
  description = "IDs of created instances"
  value       = aws_instance.web_server.*.id  # 🌟 Lists all IDs
}
```
**Command**: `terraform output`

### 🧩 **Module Structure**
```
project/
├── main.tf               # 📦 Defines modules
├── variables.tf
├── outputs.tf
└── modules/
    └── ec2_instances/
        ├── main.tf       # 🔧 Module resources
        ├── variables.tf
        └── outputs.tf
```

**Usage in main.tf**:
```hcl
module "ec2_instances" {
  source = "./modules/ec2_instances"
  instance_count = 3
  instance_type = "t3.micro"
}
```

## ⚡ **Provisioners**

### 🕒 **Provisioner Types**
- **Creation-Time**: Executes during resource creation
- **Destroy-Time**: Executes during destruction (`when = destroy`)

### 📁 **File Provisioner**
```hcl
resource "aws_instance" "server" {
  provisioner "file" {
    source      = "conf/myapp.conf"
    destination = "/etc/myapp.conf"
    
    connection {
      type        = "ssh"
      user        = "ec2-user"
      private_key = file("key.pem")
      host        = self.public_ip
    }
  }
}
```

### 💻 **Local-Exec Provisioner - Practical Example**

**📁 Project Structure**:
```
├── main.tf
├── upload_to_s3.sh
└── IaC/
    └── dsa_iac_deploy/
        ├── app.py
        ├── requirements.txt
        └── templates/
```

**🔧 Terraform Implementation**:
```hcl
resource "aws_s3_bucket" "dsa_bucket_flask" {
  bucket = "unique-bucket-name" 
  force_destroy = true  # 🗑️ Allows destroying non-empty bucket
  
  tags = {
    Name        = "DSA Bucket"
    Environment = "Lab4"
  }

  # 📤 Upload during creation
  provisioner "local-exec" {
    command = "chmod +x ${path.module}/upload_to_s3.sh && ${path.module}/upload_to_s3.sh"
  }

  # 🧹 Cleanup during destruction
  provisioner "local-exec" {
    when    = destroy
    command = "aws s3 rm s3://unique-bucket-name --recursive"
  }
}
```

**📜 upload_to_s3.sh Script**:
```bash
#!/bin/bash
# 📂 Recursive upload to S3
aws s3 cp IaC/dsa_iac_deploy/ s3://unique-bucket-name/ --recursive

# 🗂️ Structure uploaded:
# IaC/dsa_iac_deploy/
# ├── app.py              # 🐍 Flask application
# ├── requirements.txt    # 📦 Python dependencies
# └── templates/          # 🎨 HTML templates
#     ├── index.html
#     └── results.html
```

### 🌐 **Remote-Exec Provisioner**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-*****************"
  instance_type = "t3.micro"
  
  provisioner "remote-exec" {
    inline = [
      "sudo yum update -y",
      "sudo yum install httpd -y", 
      "sudo systemctl start httpd",
      "sudo bash -c 'echo Web Server with Terraform > /var/www/html/index.html'"
    ]
    
    connection {
      type        = "ssh"
      user        = "ec2-user"
      private_key = file("key.pem")
      host        = self.public_ip
    }
  }
}
```

## 🔐 **Security Groups**

### 🛡️ **Security Configuration**
```hcl
resource "aws_security_group" "web_sg" {
  name        = "allow-http-ssh"
  description = "Allows HTTP and SSH"
  
  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # 🌍 Any IP
  }
  
  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp" 
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 65535  # 📡 All output ports
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

## ☁️ **User Data as Alternative**

### 📜 **Initialization Script**
```hcl
resource "aws_instance" "server" {
  ami           = "ami-*****************"
  instance_type = "t3.micro"
  
  user_data = <<-EOF
              #!/bin/bash
              sudo yum update -y
              sudo yum install httpd -y
              sudo systemctl start httpd
              sudo systemctl enable httpd
              sudo bash -c 'echo Web Server with Terraform > /var/www/html/index.html'
              EOF
}
```

## 🏗️ **ECS Cluster Creation**

### 🔧 **Main Components**
- **`aws_ecs_cluster`**: Defines the cluster
- **`aws_ecs_task_definition`**: Configures tasks
- **`aws_ecs_service`**: Manages the service
- **Application Load Balancer (ALB)**: `aws_lb` + `aws_lb_target_group` + `aws_lb_listener`

### 💡 **Considerations**
- **Health checks** integrate with CloudWatch
- **IAM roles** can be created via Terraform
- **Security groups** need specific configuration
- **Cost**: Most expensive part of infrastructure
- **Auto-termination**: Important for cost control

## 🌍 **Availability Zones**

### 📍 **High Availability**
```hcl
# constants.tf
variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-2a", "us-east-2b", "us-east-2c"]
}
```

**Benefits**:
- 🛡️ Fault tolerance
- 🔧 Maintenance without downtime
- 📈 Load distribution

## 📊 **Terraform Graph**

### 🔍 **Dependency Visualization**
```bash
terraform graph | dot -Tpng > graph.png  # 🖼️ Generates image
```

**Online tool**: http://webgraphviz.com/

## 🐳 **Practical Example with Docker**

### 🚀 **Complete Execution Flow**
```bash
# 📂 Navigate to module directory
cd devops-iac/

# 1. 🏗️ Build Docker image
docker build -t terraform-image:lab .

# 2. 🐳 Run container
docker run -dit --name dsa-lab terraform-image:lab

# 3. 🔍 Access container
docker exec -it dsa-lab /bin/bash

# 4. ☁️ Configure AWS
aws configure
# AWS Access Key ID: ********************
# AWS Secret Access Key: ***************************************
# Region: us-east-2

# 5. ✅ Test AWS configuration
aws s3 ls | awk '{print $NF}'

# 6. 🔧 Check Terraform version
terraform version

# 7. 📁 Navigate to laboratory
cd lab

# 8. 🏁 Initialize Terraform
terraform init

# 9. 🚀 Apply configuration
terraform apply

# 10. 💥 Destroy resources (when needed)
terraform destroy
```

### 🔎 **Useful AWS Commands**
```bash
# 📋 List Amazon Linux 2 AMIs
AWS_PAGER="" aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" \
  --query 'Images[?ImageLocation.contains(@, `amazon`)].{Name:Name,ID:ImageId}' \
  --output table

# 🖥️ Create instance via CLI
aws ec2 run-instances \
  --image-id ami-***************** \
  --instance-type t3.micro \
  --region us-east-2 \
  --count 1
```

## 💡 **HCL - HashiCorp Configuration Language**

### 🌟 **Main Characteristics**
- ✅ **Readability**: Easy for humans to read and write
- 🔄 **JSON Compatibility**: Bidirectional conversion with JSON
- 🏷️ **Data Types**: Strings, numbers, booleans, lists, maps
- 🔗 **Interpolation**: References values between blocks
- 🧩 **Modularity**: Configuration reuse
- 🔧 **Expressiveness**: Supports operations and functions

### 🎯 **Best Practices with Provisioners**
- ⚠️ **Last alternative**: Use only when no native option exists
- 🔒 **Error handling**: Always validate permissions and dependencies
- 📝 **Robust scripts**: Include checks and logs
- 🧹 **Cleanup**: Implement destruction provisioners when necessary

