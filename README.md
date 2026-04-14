<!-- # Local AI Chatbot Guide

A step-by-step guide to run a local AI chatbot using Ollama, Docker, and Open WebUI on Raspberry Pi. -->
<p align="center"><a href="https://github.com/coayo-x/Local-AI-Chatbot-Guide/blob/main/README.md#setup"><img alt="Local AI Chatbot Guide. A step-by-step guide to run a local AI chatbot using Ollama, Docker, and Open WebUI on Raspberry Pi." src="./image/repo-title.png" /></a></p>


## Overview

Set up a local AI chatbot with a web interface running on your Raspberry Pi.

## Requirements

- Raspberry Pi 4 or 5
- 64-bit Raspberry Pi OS (Debian-based)
- Internet connection
- 4 GB RAM minimum
- 8 GB RAM recommended for better performance
- At least 8 GB of free storage recommended

## Stack

- Ollama
- Docker
- Open WebUI
- Model: `gemma:2b` (or any Ollama-supported model)

## Notes

- This guide is for Raspberry Pi 4/5 running 64-bit Raspberry Pi OS.
- `gemma:2b` is one of the lighter models, but it may still feel slow on a Pi 4.
- Larger models may run poorly or fail if your Pi does not have enough RAM.
- Pulling a model can take time and storage space depending on your internet speed and available disk space.
- This guide uses a fixed Open WebUI image tag for better stability instead of a moving `main` tag.
- Example fixed image tag used in this guide: `git-a382e82`

## Setup

### 1. Check system architecture

Run:

```bash
getconf LONG_BIT
uname -m
```

Expected output:

- `64`
- `aarch64`

If your system is not 64-bit, stop here and install a 64-bit version of Raspberry Pi OS before continuing.

### 2. Update system

```bash
sudo apt update
sudo apt upgrade -y
```

### 3. Install Ollama

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

This may take a while depending on your internet speed and storage.

Confirm the model is available:

```bash
ollama list
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

If you do not get a response, start Ollama manually:

```bash
ollama serve
```

Only do this if Ollama is not already running automatically.  
If you start it this way, keep that terminal open while using Open WebUI.

### 4. Install Docker

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

Optional: add your user to the Docker group if you want to run Docker commands without `sudo`

```bash
sudo usermod -aG docker $USER
newgrp docker
```

`newgrp docker` may work right away, but logging out and back in is more reliable before continuing.

If you skip this step, use `sudo` with all Docker commands below.

### 5. Run Open WebUI

Set the fixed image tag used in this guide:

```bash
export OPEN_WEBUI_IMAGE=ghcr.io/open-webui/open-webui:git-a382e82
```

Pull the image:

```bash
docker pull $OPEN_WEBUI_IMAGE
```

Run Open WebUI:

```bash
docker run -d \
--network=host \
-v open-webui:/app/backend/data \
-e OLLAMA_BASE_URL=http://127.0.0.1:11434 \
--name open-webui \
--restart always \
$OPEN_WEBUI_IMAGE
```

Check that the container is running:

```bash
docker ps
```

If you did not add your user to the Docker group, use `sudo` with the Docker commands above.

## Access

Find your Raspberry Pi IP:

```bash
hostname -I
```

Open your browser:

- `http://localhost:8080`
- `http://<your-raspberry-pi-ip>:8080`

`localhost:8080` works only on the Raspberry Pi itself.  
Use the Raspberry Pi IP from another device on the same network.

If the page opens and shows the account creation screen, the setup worked.

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

If you did not add your user to the Docker group, use `sudo` with these commands.

## Troubleshooting

### Open WebUI does not connect to Ollama

Check that Ollama is running:

```bash
curl http://127.0.0.1:11434/api/tags
```

If this does not return a response, Ollama is not available yet.

Start Ollama manually if needed:

```bash
ollama serve
```

Only use `ollama serve` if Ollama is not already running automatically.

You can also check Open WebUI logs:

```bash
docker logs open-webui
```

### Port conflict

This guide uses `--network=host`, so Open WebUI uses port `8080` directly on the Raspberry Pi.

If port `8080` is already in use, find what is using it:

```bash
sudo ss -ltnp | grep :8080
```

Then stop or reconfigure that service, and run the original Open WebUI command again.

### Container name already in use

If you get an error saying the container name is already in use, remove the old container:

```bash
docker rm -f open-webui
```

Then run the Open WebUI command again.

### Permission issues

```bash
sudo usermod -aG docker $USER
newgrp docker
```

If Docker still fails, log out and log back in.

### Download or install commands fail

If `curl` or Docker install commands fail, check:

- Internet connection
- DNS/network settings
- System date and time

## Update Open WebUI

To update later, choose another valid fixed tag from the Open WebUI container registry tags list.

You can find new fixed tags on the package page for `ghcr.io/open-webui/open-webui` in GitHub Container Registry. Open the package page, check the available tags, and choose a newer fixed tag.

Then change the image tag, pull it, and use the same `docker run` command from Step 5 again.

Example:

```bash
export OPEN_WEBUI_IMAGE=ghcr.io/open-webui/open-webui:<new-fixed-tag>
docker stop open-webui
docker rm open-webui
docker pull $OPEN_WEBUI_IMAGE
```

Then run the same Open WebUI command from Step 5.

## Done

Local AI chatbot is running.

Access it in your browser and manage it with Docker.

---

## Contact

If you have questions, run into errors, or need help with this setup, feel free to contact:

[![Email](https://img.shields.io/badge/Email-Contact%20Me-0A66C2?style=for-the-badge&logo=gmail&logoColor=white)](mailto:Local-AI-Chatbot-Guide-repo@coayo.com)

---
