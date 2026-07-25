# DEPLOYMENT.md

## TaskHub AWS EC2 Deployment Guide

### Prerequisites

Before deployment, ensure the following requirements are met:

* AWS account
* Running EC2 Ubuntu instance
* Docker installed
* Docker Compose installed
* Git installed
* Docker Hub account
* SSH key pair (`.pem` file)
* Security Group configured to allow:

  * SSH (22)
  * Frontend (3005)
  * Backend API (4000)

---

# Step 1: Launch an EC2 Instance

1. Log in to the AWS Management Console.
2. Navigate to **EC2**.
3. Launch a new Ubuntu instance.
4. Download the generated key pair (`.pem`).
5. Configure the Security Group with the following inbound rules:

| Port | Purpose     |
| ---- | ----------- |
| 22   | SSH         |
| 3005 | Frontend    |
| 4000 | Backend API |

---

# Step 2: Connect to the EC2 Instance

Move the PEM file into the SSH directory:

```bash
mv /mnt/c/Users/HP/Downloads/demo-pem.pem ~/.ssh/
chmod 400 ~/.ssh/demo-pem.pem
```

Connect via SSH:

```bash
ssh -i ~/.ssh/demo-pem.pem ubuntu@32.197.106.47
```

---

# Step 3: Install Docker

Update Ubuntu packages:

```bash
sudo apt update
```

Install Docker:

```bash
sudo apt install docker.io -y
```

Enable Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Add the current user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

Log out and reconnect to apply the new group membership.

Verify installation:

```bash
docker --version
```

---

# Step 4: Install Docker Compose

Install the Docker Compose plugin:

```bash
sudo apt install docker-compose-v2 -y
```

Verify installation:

```bash
docker compose version
```

---

# Step 5: Upload the Docker Compose File

From the local machine:

```bash
scp -i ~/.ssh/demo-pem.pem compose.aws.yml ubuntu@32.197.106.47:~
```

Verify that the file exists on the EC2 instance:

```bash
ls
```

---

# Step 6: Deploy the Application

Start the application:

```bash
docker compose -f compose.aws.yml up -d
```

Check running containers:

```bash
docker ps
```

---

# Step 7: Verify Deployment

Check the container logs if necessary:

```bash
docker compose -f compose.aws.yml logs
```

Check a specific service:

```bash
docker compose -f compose.aws.yml logs frontend
```

```bash
docker compose -f compose.aws.yml logs backend
```

```bash
docker compose -f compose.aws.yml logs mongodb
```

---

# Step 8: Access the Application

Frontend:

```
http://32.197.106.47:3005
```

Backend API:

```
http://32.197.106.47:4000
```

---

# Useful Docker Commands

Start services:

```bash
docker compose -f compose.aws.yml up -d
```

Stop services:

```bash
docker compose -f compose.aws.yml down
```

Restart services:

```bash
docker compose -f compose.aws.yml restart
```

View running containers:

```bash
docker ps
```

View all containers:

```bash
docker ps -a
```

View logs:

```bash
docker compose -f compose.aws.yml logs
```

Remove unused Docker resources:

```bash
docker system prune -f
```

---

# Troubleshooting

### Unable to SSH into the EC2 instance

* Verify that the correct `.pem` file is used.
* Ensure port 22 is open in the EC2 Security Group.
* Confirm the public IP address is correct.

### Frontend is inaccessible

* Verify that port 3005 is open in the Security Group.
* Confirm the frontend container is running:

```bash
docker ps
```

* Inspect the frontend logs:

```bash
docker compose -f compose.aws.yml logs frontend
```

### Backend cannot connect to MongoDB

* Verify the `MONGODB_URI` in `compose.aws.yml`.
* Confirm that the MongoDB container is running:

```bash
docker ps
```

* Check MongoDB logs:

```bash
docker compose -f compose.aws.yml logs mongodb
```

### Docker image pull fails

If Docker reports:

```
pull access denied
```

* Verify the Docker Hub repository name and tag.
* Confirm that the images have been pushed successfully.
* If the repository is private, authenticate using:

```bash
docker login
```

before redeploying the application.
