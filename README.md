# Local AI Chatbot Guide

A step-by-step guide to run a local AI chatbot using Ollama, Docker, and Open WebUI on Raspberry Pi.

## Overview

Set up a local AI chatbot with a web interface running on your Raspberry Pi.

## Requirements

- Raspberry Pi 4 or 5
- 64-bit Raspberry Pi OS (Debian-based)
- Internet connection
- 4 GB RAM minimum
- 8 GB RAM recommended for better performance

## Stack

- Ollama
- Docker
- Open WebUI
- Model: `gemma:2b` (or any Ollama-supported model)

## Notes

- This guide assumes you are using a 64-bit Raspberry Pi OS on a Raspberry Pi 4 or 5.
- `gemma:2b` is one of the lighter models, but it can still feel slow on a Pi 4.
- Larger models may run poorly or fail if your Pi does not have enough RAM.

## Setup

### 1. Update system

```bash
sudo apt update
sudo apt upgrade
```

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

Press `Ctrl + C` to stop the chat.

Check that Ollama is running:

```bash
curl http://127.0.0.1:11434/api/tags
```

If you get a response, Ollama is available and ready for Open WebUI.

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

If Docker commands still fail, log out and log back in.

### 4. Run Open WebUI

This guide uses the ARM64 image for Raspberry Pi 4/5 on 64-bit Raspberry Pi OS.

```bash
docker run -d \
--network=host \
-v open-webui:/app/backend/data \
-e OLLAMA_BASE_URL=http://127.0.0.1:11434 \
--name open-webui \
--restart always \
ghcr.io/open-webui/open-webui:main-arm64
```

## Access

Find your Raspberry Pi IP:

```bash
hostname -I
```

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

### Open WebUI does not connect to Ollama

Check that Ollama is running:

```bash
curl http://127.0.0.1:11434/api/tags
```

If this does not return a response, Ollama is not available yet.

You can also check Open WebUI logs:

```bash
docker logs open-webui
```

### Port conflict

If port `8080` is already in use, use the command below instead.

This command replaces the previous `docker run` command completely.

```bash
docker run -d \
-p 8081:8080 \
-v open-webui:/app/backend/data \
-e OLLAMA_BASE_URL=http://127.0.0.1:11434 \
--name open-webui \
--restart always \
ghcr.io/open-webui/open-webui:main-arm64
```

Then open:

* `http://localhost:8081`
* `http://<your-raspberry-pi-ip>:8081`

### Permission issues

```bash
sudo usermod -aG docker $USER
newgrp docker
```

If needed, log out and log back in.

## Update Open WebUI

```bash
docker stop open-webui
docker rm open-webui
docker pull ghcr.io/open-webui/open-webui:main-arm64
docker run -d \
--network=host \
-v open-webui:/app/backend/data \
-e OLLAMA_BASE_URL=http://127.0.0.1:11434 \
--name open-webui \
--restart always \
ghcr.io/open-webui/open-webui:main-arm64
```

## Done

Local AI chatbot is running.

Access it in your browser and manage it with Docker.

---