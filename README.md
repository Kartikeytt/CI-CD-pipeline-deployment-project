  EC2 Deployment Guide - Node.js Todo App

This repository features a fully automated, single-file deployment solution for setting up and running your Node.js Todo application on an AWS EC2 instance using Docker.

---

## 📄 Automated Deployment 

Save the entire block of code below into a file named on your EC2 instance or paste it directly inside your Jenkins pipeline script runner.

```bash
#!/bin/bash
# ==============================================================================
# Node.js Todo App - Single-File EC2 Deployment Script
# Credits: Based on DevOps architectures by Shubham Londhe (@londheshubham153)
# Maintained by: Kartikey Tiwari
# ==============================================================================

# Exit immediately if a command exits with a non-zero status
set -e

echo "=================================================="
echo "🚀 Starting Full Node.js App Deployment on EC2..."
echo "=================================================="

# 1. UPDATE SYSTEM PACKAGES
echo "⚙️ Updating system package repositories..."
sudo apt-get update -y

# 2. INSTALL DOCKER ENGINE (If missing)
if ! command -v docker &> /dev/null; then
    echo "🐳 Installing Docker Engine and components..."
    sudo apt-get install -y docker.io
    sudo systemctl start docker
    sudo systemctl enable docker
else
    echo "🐳 Docker is already installed. Skipping installation."
fi

# 3. CONFIGURE USER PERMISSIONS (Fixes permission denied errors)
echo "🔒 Configuring non-root group privileges for Docker..."
sudo groupadd docker 2>/dev/null || true
sudo usermod -aG docker \$USER || true
# If running via a Jenkins build execution block, grant permissions to Jenkins too
if id "jenkins" &>/dev/null; then
    sudo usermod -aG docker jenkins || true
fi

# 4. CREATE MODERN DOCKERFILE DYNAMICALLY
echo "📄 Generating production Node.js 20 Dockerfile..."
cat << 'EOF' > Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 8000
CMD ["node", "app.js"]
EOF

# 5. BUILD DOCKER IMAGE WITHOUT CACHE ISSUES
echo "🏗️ Building the production container image (todo-app)..."
sudo docker build --no-cache . -t todo-app

# 6. CLEAN UP OLD CRASHED / RUNNING CONTAINERS
echo "🧹 Cleaning up pre-existing container runtimes..."
sudo docker stop todo-app-container || true
sudo docker rm todo-app-container || true

# 7. RUN THE LIVE APPLICATION CONTAINER
echo "🌐 Launching live background container on Port 8000..."
sudo docker run -d --name todo-app-container -p 8000:8000 todo-app

# 8. VERIFY RUNTIME STATUS & LOGGING
echo "⏱️ Waiting briefly for service initiation checks..."
sleep 3

echo "=================================================="
echo "📊 CONTAINER DEPLOYMENT CHECKPOINT:"
echo "=================================================="
sudo docker ps -a

echo "=================================================="
echo "📝 RECENT APPLICATION LOG OUTPUT:"
echo "=================================================="
sudo docker logs todo-app-container

echo "=================================================="
echo "🎉 SUCCESS! Web application is actively live at:"
echo "🔗 http://\$(curl -s ifconfig.me):8000"
echo "⚠️ Make sure Port 8000 is open in your AWS EC2 Security Group Inbound Rules."
echo "=================================================="
```

---

## 🚀 How to Deploy on EC2

Follow these simple commands on your target server instance terminal:

```bash
# 1. Create the shell file
nano deploy.sh

# [Paste the script block code above into the terminal window, save, and exit]

# 2. Grant executable runtime permissions to the script
chmod +x deploy.sh

# 3. Execute the full deployment lifecycle
./deploy.sh
```

---

## 🔒 Post-Deployment Security Rules

To ensure your web application is reachable over the open public internet:
1. Open your **AWS EC2 Web Console**.
2. Navigate to your target instance and select its assigned **Security Group**.
3. Edit the **Inbound Rules** configuration settings.
4. Add a new rule: **Custom TCP**, Port range: `8000`, Source: `0.0.0.0/0` (Anywhere).

---

## 📜 Credits and Acknowledgments

* **DevOps Architecture Framework:** Engineered around training architectures designed by **Shubham Londhe** (GitHub: [@londheshubham153](https://github.com)).
