# [decyphertek.io](https://decyphertek.io/)
Decoding Technology.
* How to create an concise sysadmin AI system.
* LLM can ping questions to specialized machine learning models.
* Everything is off until a question is asked that activates a specialized machine learning model.
* LPIC reference model, ability to run bash , ansible , opentofu , python , automatically. 

1. Kubernetes:
--------------
```
curl -sfL https://get.k3s.io | sh - 
# Check for Ready node, takes ~30 seconds 
sudo k3s kubectl get node
```

2. Kubeflow:
------------
```
sudo snap install microk8s --classic
sudo usermod -a -G microk8s $USER
newgrp microk8s
sudo chown -f -R $USER ~/.kube
# Modified - Delayed install metallb and ingress , so it works on startupscript, so doesnt delete anything.
microk8s enable dns ingress cert-manager hostpath-storage rbac
microk8s status --wait-ready
sudo snap install juju --classic 
mkdir -p ~/.local/share
microk8s config | juju add-k8s my-k8s --client
juju bootstrap my-k8s uk8sx
juju add-model kubeflow
sudo sysctl fs.inotify.max_user_instances=1280
sudo sysctl fs.inotify.max_user_watches=655360
sudo sh -c 'echo "fs.inotify.max_user_instances=1280" >> /etc/sysctl.conf'
sudo sh -c 'echo "fs.inotify.max_user_watches=655360" >> /etc/sysctl.conf'
juju deploy kubeflow --trust --channel=latest/stable
sudo systemctl enable snap.microk8s.daemon-containerd
sudo systemctl enable snap.microk8s.daemon-kubelite
sudo systemctl enable snap.microk8s.daemon-cluster-agent
sudo systemctl enable snap.microk8s.daemon-k8s-dqlite
sudo systemctl enable snap.microk8s.daemon-apiserver-kicker
juju status --watch 5s
```

3. Open Web UI:
---------------
* I may need Ollama3 added to the docker-compose.yml , still in Dev.
* Need a GPU or Very slow during testing. 
~~~
vim docker-compose.yml

services:

  ingress:
    image: nginx:latest
    container_name: nginx-reverse-proxy
    restart: always
    ports:
      - 9443:9443
      - 443:443
    volumes:
      - nginx_conf:/etc/nginx/conf.d/
      - nginx_key:/etc/ssl/private/
      - nginx_cert:/etc/ssl/certs/
      - nginx_log:/var/log/
    networks:
      decyphertek-net:
        ipv4_address: 172.19.0.19

  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    expose:
      - "9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    networks:
      decyphertek-net:
        ipv4_address: 172.19.0.20

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: always
    expose:
      - "8080"
    extra_hosts:
      - "host.docker.internal:host-gateway"
    volumes:
      - /home/adminotaur/.docker:/app/backend/data
    networks:
      decyphertek-net:
        ipv4_address: 172.19.0.21

volumes:
  open-webui:
  nginx_conf:
  nginx_key:
  nginx_cert:

networks:
  decyphertek-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.19.0.0/16

docker compose up -d
# need default.conf
# openssl self signed certs
# .env
~~~

4. Brainboard ( Terraform )
---------------------------
* [https://www.brainboard.co/](https://www.pluralith.com/)

5. Theory
--------
* AI should simulate brain function. The brain optimizes energy consumption and this can be achieved with Kubernetes.

6. Cloud Theory:
-----------------
```
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
}

resource "aws_eks_cluster" "my_cluster" {
  name     = "my-cluster"
  role_arn = aws_iam_role.eks_role.arn
  vpc_config {
    subnet_ids = [aws_subnet.public.id]
  }
}

resource "aws_iam_role" "eks_role" {
  name = "eks_role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Principal = {
        Service = "eks.amazonaws.com"
      }
      Effect    = "Allow"
      Sid       = ""
    }]
  })
}

resource "aws_eks_node_group" "my_node_group" {
  cluster_name    = aws_eks_cluster.my_cluster.name
  node_group_name = "my-node-group"
  node_role_arn   = aws_iam_role.node_role.arn
  subnet_ids      = [aws_subnet.public.id]

  scaling_config {
    desired_size = 2
    max_size     = 3
    min_size     = 1
  }
}

resource "aws_iam_role" "node_role" {
  name = "node_role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
      Effect    = "Allow"
      Sid       = ""
    }]
  })
}
```
