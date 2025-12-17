Deploying the Swiggy Clone App with Terraform, Kubernetes and GitHub Actions.


[A] Let's use Terraform to create an EC2 instance for EC2 Runner, Docker and SonarQube
Content Of main.tf
    ```bash

            resource "aws_instance" "web" {
            ami                    = "ami-0287a05f0ef0e9d9a"      
            instance_type          = "t3.medium"
            key_name               = "newkey.pem"              
            vpc_security_group_ids = [aws_security_group.GitHubAction-VM-SG.id]
            user_data              = templatefile("./install.sh", {})

            tags = {
                Name = "GitHubAction-SonarQube"
            }

            root_block_device {
                volume_size = 40
                }
            }

            resource "aws_security_group" "GitHubAction-VM-SG" {
            name        = "GitHubAction-VM-SG"
            description = "Allow TLS inbound traffic"

            ingress = [
                for port in [22, 80, 443, 8080, 9000, 3000] : {
                description      = "inbound rules"
                from_port        = port
                to_port          = port
                protocol         = "tcp"
                cidr_blocks      = ["0.0.0.0/0"]
                ipv6_cidr_blocks = []
                prefix_list_ids  = []
                security_groups  = []
                self             = false
                }
            ]

            egress {
                from_port   = 0
                to_port     = 0
                protocol    = "-1"
                cidr_blocks = ["0.0.0.0/0"]
            }

            tags = {
                Name = "GitHubAction-VM-SG"
                }
            }

    ```

2 -- Content Of provider.tf
        ```bash
            terraform {
            required_providers {
                aws = {
                source = "hashicorp/aws"
                version = "6.20.0"
                    }
                }
            }
        ```
**configure the aws provider:**
    ```bash

            provider "aws" {
            region = "us-east-1"     
            }
    ```
**Step 2: Clone the Code:**

- Update all the packages and then clone the code.
- Clone your application's code repository onto the EC2 instance:
    
    ```bash
    git clone https://github.com/WasimHannure/swiggy-app.git
    ```

**Step 3: Content of install.sh:**
        ```bash

            #!/bin/bash
            sudo apt update -y
            wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
            echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
            sudo apt update -y
            sudo apt install temurin-17-jdk -y
            /usr/bin/java --version
        ```

**Step 3: Install Docker and Run the App Using a Container:**

- Set up Docker on the EC2 instance:
    
    ```bash
    
    sudo apt-get update
    sudo apt-get install docker.io -y
    sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
    newgrp docker
    sudo chmod 777 /var/run/docker.sock
    ```
**Phase 2: Security**

1. **Install SonarQube and Trivy:**
    - Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.
        
        sonarqube
        ```
        docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
        ```
        
        
        To access: 
        
        publicIP:9000 (by default username & password is admin)
        
        To install Trivy:
        ```
        sudo apt-get install wget apt-transport-https gnupg lsb-release
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
        echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
        sudo apt-get update
        sudo apt-get install trivy        
        ```
        
        to scan image using trivy
        ```
        trivy image <imageid>
        ```
        
        
2. **Integrate SonarQube and Configure:**
    - Integrate SonarQube with your CI/CD pipeline.
    - Configure SonarQube to analyze code for quality and security issues.

**Install unzip:**
    ```bash
    sudo apt-get install unzip
    ```



4 -- Terraform Commands
terraform init, terraform plan, terraform apply -auto-approve


**Create EKS**
    - install kubectl on ec2
    ```bash
    
        sudo apt update
        sudo apt install curl
        curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        kubectl version --client
    ```

2 -- Install AWS CLI
    ```bash

        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
        sudo apt install unzip
        unzip awscliv2.zip
        sudo ./aws/install
        aws --version
    ```

3 -- Installing  eksctl
    ```bash

            curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
            cd /tmp
            sudo mv /tmp/eksctl /bin
            eksctl version
    ```

4 -- Setup Kubernetes using eksctl
    ```bash

            eksctl create cluster --name my-cluster \
            --region us-east-1 \
            --node-type t2.small \
            --nodes 3 \
    ```


5 -- Verify Cluster with below command
    ```bash

            kubectl get nodes
            kubectl get all
    ```

[D] Verify CICD Pipeline through gitbash
    ```bash

            git config --global user.name "Your.Name"
            git config --global user.email "your.email@gmail.com"
            git clone https://github.com/wasimhannure/swiggy-clone-b
            git add .
            git commit -m "Changed Banner"
            git push -u origin main  //after giving this command it may ask you to provide your github credentials
    ```

[E] Cleanup
kubectl get all    ///It will show all the deployment & services.
kubectl delete deployment.apps/swiggy-app
kubectl delete service/swiggy-app
eksctl delete cluster my-cluster --region us-east-1

docker ps -a
docker stop xxx
docker rm xxx

terraform destroy








