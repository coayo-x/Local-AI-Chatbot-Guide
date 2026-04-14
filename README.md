# Local AI Chatbot Guide

A step-by-step guide to run a local AI chatbot using Ollama, Docker, and Open WebUI on Raspberry Pi.

## Overview

Set up a local AI chatbot with a web interface running on your Raspberry Pi.

## Requirements

- Raspberry Pi (4 or 5 recommended)
- Internet connection

## Stack

- Ollama
- Docker
- Open WebUI
- Models: `gemma:2b` (or any Ollama-supported model)

## Setup

### 1. Update system

```bash
sudo apt update
sudo apt upgrade
````

### 2. Install Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Check installation:

```bash
ollama --version
```

Pull a model:

```bash
ollama pull gemma:2b
```

Test the model:

```bash
ollama run gemma:2b
```

### 3. Install Docker

Install required packages:

```bash
sudo apt-get install -y ca-certificates curl gnupg
```

Create keyring directory:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Add Docker GPG key:

```bash
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add Docker repository:

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/debian \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Update packages:

```bash
sudo apt-get update
```

Install Docker:

```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

Verify Docker:

```bash
sudo docker run hello-world
```

Optional: add your user to Docker group

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### 4. Run Open WebUI

```bash
docker run -d \
--network=host \
-v open-webui:/app/backend/data \
-e OLLAMA_BASE_URL=http://127.0.0.1:11434 \
--name open-webui \
--restart always \
ghcr.io/open-webui/open-webui:main
```

## Access

Open your browser:

* `http://localhost:8080`
* `http://<your-raspberry-pi-ip>:8080`

Create an account. The first account becomes the admin.

## Container Management

Stop:

```bash
docker stop open-webui
```

Start:

```bash
docker start open-webui
```

Logs:

```bash
docker logs open-webui
```

## Troubleshooting

### Port conflict

```bash
-p 8081:8080
```

### Permission issues

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### ARM64 (Raspberry Pi 4/5)

```bash
ghcr.io/open-webui/open-webui:main-arm64
```

## Update Open WebUI

```bash
docker stop open-webui
docker rm open-webui
docker pull ghcr.io/open-webui/open-webui:main
docker run -d \
--network=host \
-v open-webui:/app/backend/data \
-e OLLAMA_BASE_URL=http://127.0.0.1:11434 \
--name open-webui \
--restart always \
ghcr.io/open-webui/open-webui:main
```

## Done

Local AI chatbot is running.

Access it in your browser and manage it with Docker.

```

---
