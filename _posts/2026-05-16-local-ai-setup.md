---
title: "Setting Up a Local AI Stack on Arch Linux for CTF and Ethical Hacking"
date: 2026-05-16 23:00:00 +0530
layout: post
categories: [AI, local AI setup]
tags: [arch linux, ollama, open-webui, local ai, nvidia, docker]
---

So I've been getting into CTF challenges and ethical hacking lately, and I wanted a local AI assistant that I could use for security research without worrying about sending sensitive data to the cloud and also you know public  use models dont give enough context or commands for ctf challenges by default because of its security issues and concerns. Here's how I set everything up on my Arch Linux machine with an RTX 4050.

## My Setup

Before getting into it, here's what I'm working with:

- **LAPTOP:** ASUS Gaming V16
- **OS:** Arch Linux (KDE Plasma 6)
- **CPU:** Intel Core 5 210H
- **GPU:** NVIDIA RTX 4050 Mobile (6GB VRAM)
- **RAM:** 15.6GB

The RTX 4050 is the key piece here — 6GB of VRAM is enough to run 7-8B parameter models fully on the GPU, which makes inference actually usable.

## Picking the Right Model

I spent a while going back and forth on which model to use. I looked at a few options:

- **bartowski/Qwen2.5-7B-Instruct-GGUF** — standard, highest quality
- **richardyoung/Qwen2.5-7B-Instruct-abliterated-GGUF** — refusals removed, same base quality
- **QuantFactory/Qwen2.5-7B-Instruct-Uncensored-GGUF** — fine-tuned uncensored variant

I ended up going with the **abliterated version**. The difference between abliterated and uncensored is worth understanding — abliterated models just have the refusal direction surgically removed from the weights with minimal quality loss, while uncensored fine-tunes are retrained on often low-quality datasets which can hurt technical performance. For CTF work, you want a sharp model, not just an unrestricted one.

The quantization I went with was **Q4_K_M** — the sweet spot between quality and size for 6GB VRAM.

## Installing Ollama

Ollama makes running local models incredibly simple. Installation on Arch:

```bash
sudo pacman -S ollama
sudo systemctl enable ollama --now
```

## Downloading the Model

The `huggingface-cli` command is deprecated in newer versions of the `huggingface-hub` package. Use `hf` instead:

```bash
pip install huggingface-hub --break-system-packages
hf download richardyoung/Qwen2.5-7B-Instruct-abliterated-GGUF \
  --include "*Q4_K_M*" \
  --local-dir ~/models
```

Once downloaded, import it into Ollama:

```bash
echo "FROM /home/spirit/models/Qwen2.5-7B-Instruct-abliterated-Q4_K_M.gguf" > ~/models/Modelfile
ollama create qwen-abliterated -f ~/models/Modelfile
ollama run qwen-abliterated
```

## Getting GPU Acceleration Working

This was the trickiest part. By default Ollama was running entirely on CPU at around 7 tokens per second. The fix was installing the CUDA-enabled version of Ollama:

```bash
sudo pacman -R ollama
yay -S ollama-cuda
sudo systemctl restart ollama
```

After switching to `ollama-cuda`, the model loaded onto the GPU and speed jumped significantly. You can verify with:

```bash
ollama ps
# Should show 100% GPU instead of 100% CPU
```

## Setting Up Open WebUI

I wanted a proper chat interface instead of using the terminal every time. Open WebUI gives you a ChatGPT-like experience for your local models.

Getting it running with Docker was a bit of a journey on my system because my kernel (Linux 7.0.x) was missing some modules that Docker's networking needed. The fix was creating a Docker daemon config:

```bash
sudo mkdir -p /etc/docker
echo '{"storage-driver": "overlay2"}' | sudo tee /etc/docker/daemon.json
sudo systemctl enable docker --now
```

Then running the Open WebUI container:

```bash
docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  -v open-webui:/app/backend/data \
  ghcr.io/open-webui/open-webui:main
```

One important thing — Ollama by default only listens on localhost, which means Docker can't reach it. Fix that by editing the service:

```bash
sudo systemctl edit ollama
```

Add:

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
```

After that, Open WebUI at `http://localhost:3000` connected to Ollama perfectly.

## Adding OSINT Models

For OSINT research I also pulled down **Horus-OSINT**, which is a Llama 3 8B fine-tune specifically trained on OSINT methodology:

```bash
hf download mahmoudalyosify/Horus-OSINT \
  --include "*Q4_K_M*" \
  --local-dir ~/models/horus

echo "FROM /home/spirit/models/horus/llama-3-8b-instruct.Q4_K_M.gguf" > ~/models/horus/Modelfile
ollama create horus -f ~/models/horus/Modelfile
```

Now I can switch between models in Open WebUI depending on the task.

## Quality of Life — Start/Stop Aliases

this is little trick to make life easier, and if u are a lassy person like me.

Added these to `~/.bashrc` so I can start and stop everything with one command:

```bash
echo "alias startai='sudo systemctl start ollama && docker start \$(docker ps -aq)'" >> ~/.bashrc
echo "alias stopai='sudo systemctl stop ollama && docker stop \$(docker ps -q)'" >> ~/.bashrc
source ~/.bashrc
```

Now `startai` brings everything up and `stopai` shuts it down cleanly. 

## Final Thoughts

The whole stack works really well for CTF and security research. And the whole process of setting this up is way easier than i thought. Having a local model means no data leaves the machine, which matters when you're working with challenge files or sensitive information and need no restrictions. The GPU acceleration makes it fast enough to actually be useful — running a 7B model at 36+ tokens per second on average on a laptop GPU is pretty solid.

If you're on Arch and want to do the same, the main things to watch out for are:

1. Install `ollama-cuda` not regular `ollama` if you have an NVIDIA GPU
2. Use `hf` not `huggingface-cli` with newer versions of huggingface-hub
3. Set `OLLAMA_HOST=0.0.0.0` if you're using Docker for the web UI

Happy hacking.
