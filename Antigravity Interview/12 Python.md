# Python Interview Guide

> **Source:** [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `12-python/` folder
> **Purpose:** Master Python scripting, automation, and DevOps tooling for interviews

---

## Table of Contents

| # | Topic |
|---|-------|
| [1](#1-common-python-packages-used-by-devops-engineers) | Common Python Packages Used by DevOps Engineers |
| [2](#2-run-a-docker-container-with-python) | Run a Docker Container with Python |
| [3](#3-find-and-print-404-logs) | Find and Print 404 Logs |

---

## 1. Common Python Packages Used by DevOps Engineers

> **Q: What are some common Python packages that you use as a DevOps Engineer?**

### Short Explanation

DevOps Engineers use Python for scripting, automation, cloud interactions, and infrastructure management. A set of well-known packages simplify tasks related to OS operations, cloud APIs, configuration, monitoring, and CI/CD workflows.

### Answer

| Package | Purpose |
|---|---|
| `boto3` | AWS SDK — manage EC2, S3, Lambda, and other AWS services |
| `paramiko` | SSH client — run remote shell commands or transfer files via SFTP |
| `requests` | HTTP client — interact with REST APIs and webhooks |
| `pyyaml` | YAML parser — read/write Kubernetes manifests and config files |
| `docker` | Docker SDK — manage containers, images, and volumes programmatically |
| `kubernetes` | Kubernetes client — automate resource creation, deletion, and monitoring |
| `fabric` | High-level SSH — simplify automation tasks on remote servers |
| `pytest` | Testing framework — write automated tests for scripts and infra code |

---

### Detailed Examples

#### 1. `boto3` — AWS SDK for Python

```python
import boto3

ec2 = boto3.client('ec2')
response = ec2.describe_instances()
print(response)
```

---

#### 2. `paramiko` — SSH and Remote Command Execution

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

#### 3. `requests` — HTTP Requests

```python
import requests

response = requests.get('https://api.example.com/status')
print(response.status_code)
print(response.json())
```

---

#### 4. `pyyaml` — YAML Parsing and Generation

```python
import yaml

with open('config.yaml', 'r') as file:
    config = yaml.safe_load(file)

print(config)
```

---

#### 5. `docker` — Docker SDK for Python

```python
import docker

client = docker.from_env()
for container in client.containers.list():
    print(container.name, container.status)
```

---

#### 6. `kubernetes` — Kubernetes Python Client

```python
from kubernetes import client, config

config.load_kube_config()
v1 = client.CoreV1Api()
pods = v1.list_pod_for_all_namespaces()

for pod in pods.items:
    print(pod.metadata.name)
```

---

#### 7. `fabric` — High-Level SSH Command Execution

```python
from fabric import Connection

c = Connection('user@host')
c.run('uname -a')
```

---

#### 8. `pytest` — Python Testing Framework

```python
def add(a, b):
    return a + b

def test_add():
    assert add(2, 3) == 5
```

Run tests with:

```bash
pytest test_script.py -v
```

---

## 2. Run a Docker Container with Python

> **Q: Write a Python script to run a container using the Docker SDK. The user provides the image name as input.**

### Short Explanation

The `docker` Python SDK lets you programmatically start Docker containers. This script takes an image name as input, pulls it if not available locally, then runs the container in detached mode.

### Answer

Use `docker.from_env()` to connect to the local Docker engine, then call `containers.run()` with `detach=True`.

---

### Complete Script

```python
import docker

# Initialize Docker client
client = docker.from_env()

# Get image name from user input
image_name = input("Enter the Docker image name (e.g., nginx:latest): ").strip()

try:
    # Pull the image if not present locally
    print(f"Pulling image '{image_name}'...")
    client.images.pull(image_name)
    print(f"Image '{image_name}' pulled successfully.")

    # Run the container in the background
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

---

### Example Run

```
Enter the Docker image name (e.g., nginx:latest): alpine
```

```
Pulling image 'alpine'...
Image 'alpine' pulled successfully.
Running container from image 'alpine'...
Container started with ID: 3e1fabc2d789
```

---

### How It Works

| Step | What Happens |
|---|---|
| `docker.from_env()` | Connects to the local Docker daemon using environment variables |
| `client.images.pull()` | Pulls the image from the registry if not cached locally |
| `client.containers.run()` | Starts the container; `detach=True` runs it in the background |
| Error handling | Catches `ImageNotFound`, `APIError`, and unexpected exceptions |

> **Note:** This script requires Docker to be installed and running. The container runs with no port mapping or custom command — extend as needed.

---

## 3. Find and Print 404 Logs

> **Q: Write a Python script to fetch logs from a log URL and print all lines with `404: Not Found`.**

### Short Explanation

This script fetches a publicly available Apache log file using `requests`, parses it line by line, and filters for entries containing HTTP status code `404`.

### Answer

Use `requests.get()` to fetch the log content, split it into lines, and filter for `' 404 '`.

---

### Complete Script

```python
import requests

# Publicly available Apache log sample
log_url = (
    'https://raw.githubusercontent.com/elastic/examples/master/'
    'Common%20Data%20Formats/apache_logs/apache_logs'
)

try:
    # Fetch the log content
    response = requests.get(log_url)
    response.raise_for_status()

    logs = response.text.splitlines()

    print("Log lines with 404 Not Found:\n")

    found = False
    for line in logs:
        if ' 404 ' in line:
            print(line)
            found = True

    if not found:
        print("No 404 entries found.")

except requests.exceptions.RequestException as e:
    print(f"Error fetching logs: {e}")
```

---

### Example Matching Line

```
216.46.173.126 - - [27/May/2015:10:27:47 +0000] "GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1" 404 146
```

---

### How It Works

| Step | What Happens |
|---|---|
| `requests.get(log_url)` | Fetches the raw log file from the public URL |
| `response.raise_for_status()` | Raises an error if the HTTP response is not 2xx |
| `response.text.splitlines()` | Splits the full log text into individual lines |
| `if ' 404 ' in line` | Filters lines with ` 404 ` (spaces prevent false matches in URLs or timestamps) |
| Error handling | Catches network errors via `RequestException` |

---

### Extend the Script

```python
# Write filtered lines to a file instead of printing
with open('404_errors.log', 'w') as f:
    for line in logs:
        if ' 404 ' in line:
            f.write(line + '\n')

# Filter for other status codes
status_codes = [' 404 ', ' 500 ', ' 403 ']
for line in logs:
    if any(code in line for code in status_codes):
        print(line)
```

---

## Quick Reference Cheatsheet

```
=========================================================
         PYTHON FOR DEVOPS — INTERVIEW CHEATSHEET
=========================================================

  KEY PACKAGES
  ------------
  boto3       → AWS SDK (EC2, S3, Lambda, IAM...)
  paramiko    → SSH client, remote command execution
  requests    → HTTP GET/POST, REST APIs, webhooks
  pyyaml      → Parse/generate YAML (K8s manifests, configs)
  docker      → Docker SDK (run/stop/list containers)
  kubernetes  → K8s client (pods, deployments, services)
  fabric      → High-level SSH automation
  pytest      → Testing framework for scripts and infra

  DOCKER SDK PATTERN
  ------------------
  client = docker.from_env()
  client.images.pull(image)
  container = client.containers.run(image, detach=True)
  container.id[:12]   ← short container ID

  LOG FILTERING PATTERN
  ---------------------
  response = requests.get(url)
  response.raise_for_status()
  for line in response.text.splitlines():
      if ' 404 ' in line:
          print(line)

  YAML PATTERN
  ------------
  import yaml
  config = yaml.safe_load(open('file.yaml'))

  K8S CLIENT PATTERN
  ------------------
  config.load_kube_config()
  v1 = client.CoreV1Api()
  pods = v1.list_pod_for_all_namespaces()

=========================================================
```

---

## Interview Tips

| Do This | Not This |
|---|---|
| Name specific packages with their use case | Just say "I use Python libraries" |
| Show error handling (`try/except`) in scripts | Write scripts with no exception handling |
| Explain why `' 404 '` has spaces (avoids false matches) | Hardcode patterns without explaining the logic |
| Mention `detach=True` runs the container in the background | Forget to explain key parameters |
| Show how to extend scripts (write to file, multiple status codes) | Give only the bare minimum answer |

---

## Resources

- [boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- [paramiko Documentation](https://www.paramiko.org/)
- [requests Documentation](https://requests.readthedocs.io/)
- [PyYAML Documentation](https://pyyaml.org/wiki/PyYAMLDocumentation)
- [Docker SDK for Python](https://docker-py.readthedocs.io/)
- [Kubernetes Python Client](https://github.com/kubernetes-client/python)
- [pytest Documentation](https://docs.pytest.org/)
- [Original Source: devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide)

---

> Star this repo if it helped you prepare for your DevOps interview!
> Drop the next topic's raw notes and they will be formatted and added here.
