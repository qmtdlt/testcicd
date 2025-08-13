# 🚀 Minimal CI/CD Setup Guide

> A simple step-by-step guide to set up automated deployment with GitHub Actions for ASP.NET Core applications.

## 📋 Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup Steps](#setup-steps)
- [GitHub Actions Workflow](#github-actions-workflow)
- [Troubleshooting](#troubleshooting)
- [Known Limitations](#known-limitations)

## 🎯 Overview

This guide helps you set up a minimal CI/CD pipeline that will:
- ✅ Automatically build your ASP.NET Core application
- ✅ Deploy to your server when code is pushed to the main branch
- ✅ Support automated testing (advanced scenarios)

## ⚡ Prerequisites

Before starting, make sure you have:

- 🌐 **GitHub Account**: Access to GitHub (Gitee also supported)
- 🖥️ **Public Server**: A server with public IPv4 address
- 🔐 **SSH Keys**: Configured GitHub SSH public and private keys

## 📝 Setup Steps

### Step 1: Create GitHub Repository

1. Create a new repository on GitHub
2. Clone the repository to your development machine

### Step 2: Project Structure Setup

Create your project with the following structure:
```
📁 testcicd/
└── 📁 src/
    └── 📁 code/
        └── 📁 testcicd/
            ├── 📄 testcicd.sln
            └── 📁 testcicd/
                └── 📄 testcicd.csproj
```

> 💡 **Important**: Pay attention to your project path structure, as it will be used in the workflow configuration.

### Step 3: Create GitHub Actions Workflow

Create the workflow directory structure:

```
📁 testcicd/
├── 📁 .git/
├── 📁 .github/
│   └── 📁 workflows/
│       └── 📄 deploy.yml
```

### Step 4: Configure GitHub Secrets

Navigate to your GitHub repository settings and add the following secrets:

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `SSH_HOST` | Public IPv4 address or domain name | `192.168.1.100` or `example.com` |
| `SSH_USER` | Server login username | `root` or `ubuntu` |
| `SSH_KEY` | Your GitHub private key content | Contents of `~/.ssh/id_rsa` |

> ⚠️ **Note**: IPv6 addresses are not supported. Use IPv4 or domain names only.

![GitHub Secrets Configuration](image.png)

### Step 5: SSH Key Setup

#### On your server:
```bash
# Add your public key to authorized keys
echo "your_public_key_content" >> ~/.ssh/authorized_keys

# Set proper permissions
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

#### Test the connection from your development machine:
```bash
# Test SSH connection with private key
ssh -i ~/.ssh/github_id_rsa root@your_server_ip
```

✅ If you can connect successfully, the SSH setup is working correctly.

## 🔧 GitHub Actions Workflow

Create or edit `.github/workflows/deploy.yml` with the following content:
```yaml
name: 🚀 Build and Deploy ASP.NET Core

on:
  push:
    branches: [ main ]    # Trigger on push to main branch
  pull_request:
    branches: [ main ]    # Also trigger on pull requests

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: 📥 Checkout code
      uses: actions/checkout@v4

    - name: 🛠️ Setup .NET SDK
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '9.0.x'

    - name: 📦 Restore dependencies
      run: dotnet restore ./src/code/testcicd/testcicd.sln

    - name: 🏗️ Build application
      run: dotnet build ./src/code/testcicd/testcicd.sln --configuration Release --no-restore

    - name: 🧪 Run tests (if any)
      run: dotnet test ./src/code/testcicd/testcicd.sln --configuration Release --no-build --verbosity normal
      continue-on-error: true

    - name: 📤 Publish application
      run: dotnet publish ./src/code/testcicd/testcicd.sln --configuration Release --output ./publish --no-build

    - name: 🚀 Deploy to server
      uses: appleboy/scp-action@v0.1.7
      with:
        host: ${{ secrets.SSH_HOST }}
        username: ${{ secrets.SSH_USER }}
        key: ${{ secrets.SSH_KEY }}
        source: "publish/*"
        target: "/home/cicd/testcicd"
        strip_components: 1
```

### Step 6: Deploy and Test

1. **Commit and Push**: Commit your changes and push to GitHub
   ```bash
   git add .
   git commit -m "Add CI/CD workflow"
   git push origin main
   ```

2. **Monitor Deployment**: Check the GitHub Actions tab to see the automated execution

![GitHub Actions Execution](image-1.png)

3. **Verify on Server**: Check your server to confirm files are deployed

![Server Deployment Verification](image-2.png)

## 🔍 Troubleshooting

### Common Issues:

1. **SSH Connection Failed**
   - Verify SSH_HOST, SSH_USER, and SSH_KEY secrets
   - Test SSH connection manually
   - Check server firewall settings

2. **Build Failed**
   - Ensure .NET SDK version matches your project
   - Check for missing dependencies
   - Verify project path in workflow file

3. **Deployment Path Issues**
   - Create target directory on server: `mkdir -p /home/cicd/testcicd`
   - Check server permissions for the target directory

## ⚠️ Known Limitations

### Current Setup Limitations:

- **🔧 Manual Server Setup**: You need to manually install .NET runtime on the target server
- **🎛️ Service Management**: Setting up systemd services for background execution requires manual configuration
- **🔄 No Rollback**: This basic setup doesn't include automatic rollback on deployment failure

### Next Steps for Production:

1. **Install .NET Runtime** on your server:
   ```bash
   wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
   sudo dpkg -i packages-microsoft-prod.deb
   sudo apt-get update
   sudo apt-get install -y aspnetcore-runtime-9.0
   ```

2. **Create systemd service** for your application:
   ```bash
   sudo nano /etc/systemd/system/testcicd.service
   ```

3. **Consider GitLab** for private deployments (requires higher server specs)

---

📚 **Additional Resources:**
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [ASP.NET Core Deployment Guide](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/)
- [SSH Key Management](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
