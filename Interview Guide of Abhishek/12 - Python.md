# Python Interview Guide

> Source: [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `12-python/` folder

---

# Table of Contents

1. [Common Python Packages Used by DevOps Engineers](#1-common-python-packages-used-by-devops-engineers)
2. [Day-to-Day Python Task — Run a Docker Container](#2-day-to-day-python-task--run-a-docker-container)
3. [Find and Print 404 Logs](#3-find-and-print-404-logs)

---

# 1. Common Python Packages Used by DevOps Engineers

## Question
What are some common Python packages that you use as a DevOps Engineer?

### 📝 Short Explanation
DevOps Engineers often use Python for scripting, automation, cloud interactions, and infrastructure management. A set of well-known packages help simplify tasks related to OS operations, cloud APIs, configuration, monitoring, and CI/CD workflows.

## ✅ Answer
Some commonly used Python packages for DevOps Engineers include `boto3`, `paramiko`, `requests`, `pyyaml`, `docker`, `kubernetes`, `fabric`, and `pytest`.

### Detailed Explanation (with examples)

---

### 1. `boto3` — AWS SDK for Python
Used to automate and manage AWS services.

```python
import boto3
ec2 = boto3.client('ec2')
response = ec2.describe_instances()
print(response)
```

---

### 2. `paramiko` — SSH and Remote Command Execution
Useful for running remote shell commands or transferring files via SFTP.

```python
import paramiko
ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect(hostname='remote.server.com', username='user', password='pass')
stdin, stdout, stderr = ssh.exec_command('uptime')
print(stdout.read().decode())
ssh.close()
```

---

### 3. `requests` — HTTP Requests
Used for interacting with REST APIs and webhooks.

```python
import requests
response = requests.get('https://api.example.com/status')
print(response.status_code)
print(response.json())
```

---

### 4. `pyyaml` — YAML Parsing and Generation
Very useful when dealing with Kubernetes manifests or configuration files.

```python
import yaml
with open('config.yaml', 'r') as file:
    config = yaml.safe_load(file)
print(config)
```

---

### 5. `docker` — Docker SDK for Python
Used to manage Docker containers, images, and volumes programmatically.

```python
import docker
client = docker.from_env()
for container in client.containers.list():
    print(container.name, container.status)
```

---

### 6. `kubernetes` — Kubernetes Python Client
Helps in automating Kubernetes resource creation, deletion, and monitoring.

```python
from kubernetes import client, config
config.load_kube_config()
v1 = client.CoreV1Api()
pods = v1.list_pod_for_all_namespaces()
for pod in pods.items:
    print(pod.metadata.name)
```

---

### 7. `fabric` — High-level SSH Command Execution
Simplifies automation tasks on remote servers.

```python
from fabric import Connection
c = Connection('user@host')
c.run('uname -a')
```

---

### 8. `pytest` — Python Testing Framework
Useful for writing automated tests for infrastructure or config scripts.

```python
def add(a, b):
    return a + b

def test_add():
    assert add(2, 3) == 5
```

---

---

# 2. Day-to-Day Python Task — Run a Docker Container

## Question
Write a Python script to run a container using the Docker SDK. The user provides the image name as input.

### 📝 Short Explanation
You can use the `docker` Python SDK to programmatically start a Docker container. This script takes an image name as input, pulls the image if it doesn't exist locally, and runs a container based on it.

## ✅ Answer
Use `docker.from_env()` to connect to the local Docker engine and run the container with the provided image name.

### Complete Python Script

```python
import docker

# Initialize Docker client
client = docker.from_env()

# Get image name from user input
image_name = input("Enter the Docker image name (e.g., nginx:latest): ").strip()

try:
    # Pull the image (if not present locally)
    print(f"Pulling image '{image_name}'...")
    client.images.pull(image_name)
    print(f"Image '{image_name}' pulled successfully.")

    # Run the container
    print(f"Running container from image '{image_name}'...")
    container = client.containers.run(image_name, detach=True)
    print(f"Container started with ID: {container.id[:12]}")

except docker.errors.ImageNotFound:
    print(f"Error: Image '{image_name}' not found.")
except docker.errors.APIError as e:
    print(f"Docker API error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
```

### Example Usage

When you run the script:

```
Enter the Docker image name (e.g., nginx:latest): alpine
```

Expected output:

```
Pulling image 'alpine'...
Image 'alpine' pulled successfully.
Running container from image 'alpine'...
Container started with ID: 3e1fabc2d789
```

> This script assumes Docker is installed and the user has permission to run Docker commands. It runs the container in the background (`detach=True`) without any specific command or port mapping.

---

---

# 3. Find and Print 404 Logs

## Question
Write a Python script to fetch logs from a log website and print all logs with `404: Not Found`.

### 📝 Short Explanation
This script fetches logs from a publicly available log file, parses it line by line, and prints all entries that include the `404` HTTP status code, which typically means "Not Found".

## ✅ Answer
Use Python's `requests` library to fetch the log file from a public URL and filter for lines containing `404`.

### Complete Python Script

```python
import requests

# Publicly available Apache log sample
log_url = 'https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs'

try:
    # Fetch the log content
    response = requests.get(log_url)
    response.raise_for_status()
    logs = response.text.splitlines()

    print("Log lines with 404 Not Found:\n")

    # Search for lines with HTTP 404
    for line in logs:
        if ' 404 ' in line:
            print(line)

except requests.exceptions.RequestException as e:
    print(f"Error fetching logs: {e}")
```

### Example Matching Line from the Log:

```
216.46.173.126 - - [27/May/2015:10:27:47 +0000] "GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1" 404 146
```

### How This Script Works:
- Fetches logs using `requests.get()`
- Splits the logs line by line
- Filters for those containing ` 404 ` to avoid false matches in URLs or timestamps
- Prints all matching lines to the console

> You can modify the script to write the filtered lines to a file or analyze other HTTP status codes like `500`, `403`, etc.

---
