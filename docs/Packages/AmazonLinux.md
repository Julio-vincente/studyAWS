# Essential AWS Development Tools

Quick reference guide for installing and configuring essential development tools for Amazon Linux AWS environments.

## Packages

### AWS CLI

```bash
# AWS CLI v2 (latest)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
unzip awscliv2.zip && \
sudo ./aws/install && \
rm -rf aws awscliv2.zip
```

```bash
# Verify installation
aws --version
```

---
    
### curl

```bash
# Install curl
sudo yum install -y curl
```

```bash
# Verify installation
curl --version
```

---

### Docker

```bash
# Install and configure Docker
sudo amazon-linux-extras install docker -y && \
sudo systemctl enable --now docker && \
sudo usermod -aG docker $USER && \
newgrp docker
```

```bash
# Verify installation
docker --version
```

---

### jq

```bash
# Install jq JSON processor
sudo yum install -y jq
```

```bash
# Verify installation
jq --version
```

---

### kubectl

```bash
# Install kubectl
curl -Lo kubectl "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
chmod +x kubectl && \
sudo mv kubectl /usr/local/bin/ && \
echo 'source <(kubectl completion bash)' >>~/.bashrc
```

```bash
# Verify installation
kubectl version --client
```

```bash
# Configure kubectl for EKS
aws eks update-kubeconfig --region <region> --name <cluster-name>
```

---

## Template for Adding New Tools

```markdown
## Tool Name

```bash
# Install tool
installation commands
```

```bash
# Verify installation
verification command
```
