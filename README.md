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

