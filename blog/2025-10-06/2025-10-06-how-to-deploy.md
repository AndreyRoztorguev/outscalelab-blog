---
slug: deploy
title: How to Deploy App to AWS EC2 with Nginx
authors: [andriiroztorhuiev]
tags: [deploy, aws, nginx]
---

<!-- truncate -->

![Nginx logo](https://miro.medium.com/0*mjG1YdoT7xPcnznN.jpg)

Deploying a front-end application to AWS EC2 gives you full control over your hosting environment. This guide walks you through the entire process — from launching an instance to serving your app via Nginx.

<!-- truncate -->

## Prerequisites

Before starting, make sure you have:

- An AWS account
- A built front-end app (React, Vite, Next.js, etc.)
- Basic knowledge of the terminal
- SSH access to your EC2 instance
- Nginx installed (we’ll cover that below)

## Launch an EC2 Instance

- Go to the AWS EC2 Console
- Click Launch Instance
- Choose an Amazon Machine Image (AMI) — e.g., Amazon Linux 2 or Ubuntu.
- Choose an instance type (e.g., t2.micro for testing).
- Configure:
  - Security group:
    - Allow inbound traffic on ports 22 (SSH) and 80 (HTTP).
- Launch and download your .pem key.

### Connect to EC2 via SSH

- From your local terminal, navigate to the directory with your .pem key and connect:

```bash
chmod 400 my-key.pem
ssh -i my-key.pem ec2-user@<your-ec2-public-ip>
```

### Install Dependencies

Once connected, update packages and install Node.js, npm, and Nginx:

```bash
sudo apt update -y && sudo apt upgrade -y
sudo apt install -y nodejs npm nginx
```

### Upload Your App

- Clone from GitHub

```bash
git clone https://github.com/yourusername/your-frontend-repo.git
cd your-frontend-repo
```

- Upload via SCP (from local machine)

```bash
scp -i my-key.pem -r ./dist ec2-user@<your-ec2-ip>:/home/ec2-user
```

## Build Your App

If your app is not yet built:

```bash
npm install
npm run build
```

This will create a dist/ or build/ folder with static files.

## Configure Nginx

Open the Nginx config file:

```bash
sudo nano /etc/nginx/sites-available/default
```

Replace its contents with:

```nginx
server {
  listen 80;
  server_name _;

  root /home/ec2-user/your-frontend-repo/dist; # or build for CRA
  index index.html;

  location / {
    try_files $uri /index.html;
  }
}
```

Test and reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl enable nginx
```

## Adjust File Permissions

Make sure your app directory is accessible:

```bash
sudo chown -R www-data:www-data /home/ec2-user/your-frontend-repo
sudo chmod -R 755 /home/ec2-user/your-frontend-repo
```

## Access Your App

Open your browser and go to:

```bash
http://<your-ec2-public-ip>
```

## (Optional) Add SSL with Let’s Encrypt

Install Certbot to enable HTTPS:

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```
