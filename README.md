

# Spring Boot CI/CD Pipeline on Red Hat (RHEL)

A complete GitOps CI/CD pipeline demonstrating automated build, test, code analysis, containerization, and deployment to Kubernetes.

## 🏗️ Architecture

```
Developer → GitHub → Jenkins Pipeline → Docker Hub → ArgoCD → Kubernetes → Users
                     ↓
                SonarQube

```

---

## 🛠️ Infrastructure Setup (Red Hat Specific)

### 1️⃣ Install Jenkins

On Red Hat, we use the `dnf` package manager and the official Jenkins repo.

```bash
# Install Java 11 (Amazon Corretto or OpenJDK)
sudo dnf install java-11-openjdk-devel -y

# Add Jenkins repository for RHEL
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# Install Jenkins
sudo dnf upgrade
sudo dnf install jenkins -y

# Start and enable Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

```

### 2️⃣ Install Docker & Permissions

Red Hat uses `dnf` for Docker. Ensure you fix permissions so Jenkins can build images.

```bash
# Install Docker
sudo dnf install -y docker-ce docker-ce-cli containerd.io --allowerasing

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# CRITICAL: Fix Jenkins Permissions
sudo usermod -aG docker jenkins
sudo usermod -aG docker $USER
newgrp docker
sudo systemctl restart jenkins

# Fix workspace permission issues mentioned in troubleshooting
sudo chown -R jenkins:jenkins /var/lib/jenkins/workspace/

```

### 3️⃣ Setup Git Workspace (Red Hat Local)

Follow these steps to initialize your repository and push your project to GitHub from your Red Hat terminal:

```bash
# 1. Clone source and copy files
git clone https://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git temp_repo
cp -r temp_repo/java-maven-sonar-argocd-helm-k8s/spring-boot-app/* .
rm -rf temp_repo

# 2. Initialize your specific repo
git init
git branch -M main
git remote remove origin 2>/dev/null
git remote add origin https://github.com/bahi1023/day3.git

# 3. Clean up old Jenkins artifacts (if any)
sudo rm -rf /var/lib/jenkins/workspace/spring-boot-app

# 4. Push to GitHub
git add .
git commit -m "First push from Red Hat"
git push -u origin main

```

---

## ⚙️ Pipeline Configuration

### Jenkins Credentials

Go to **Manage Jenkins** → **Credentials** → **Global** → **Add Credentials**:

1. **sonarqube**: Secret text (Your SonarQube Token)
2. **docker-cred**: Username/Password (Docker Hub ID)
3. **github**: Secret text (Personal Access Token)

---

## 🔧 Red Hat Troubleshooting

### Firewall Configuration

RHEL has a strict firewall. You must open the ports for Jenkins, SonarQube, and ArgoCD:

```bash
# Open ports for the tools
sudo firewall-cmd --permanent --add-port=8080/tcp  # Jenkins
sudo firewall-cmd --permanent --add-port=9000/tcp  # SonarQube
sudo firewall-cmd --permanent --add-port=8081/tcp  # ArgoCD UI
sudo firewall-cmd --reload

```

### SELinux Issues

If Jenkins cannot access or edit Docker workspace, you may need to edit jenkinsfile 
you must add permission for jenkins to access add it to docker group then take it's id by the following command 
```bash
getent group docker | cut -d: -f3
```
 args '--group-add 984 -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/jenkins/workspace:/var/lib/jenkins/workspace:z' 
 /var/run/docker.sock for security reasons so that jenkins only who can access

### Docker User Fix

If your pipeline still fails with "Permission Denied" during Docker build, ensure you are using the `--user` flag in your Jenkinsfile or that the Jenkins service was fully restarted after adding it to the `docker` group.

---

 
