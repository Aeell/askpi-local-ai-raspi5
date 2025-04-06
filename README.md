# askpi-local-ai-raspi5
 Local AI Assistant in Terminal using Raspberry Pi 5 (4GB) and Ollama
# Local AI Assistant in Terminal using Raspberry Pi 5 (4GB) and Ollama

This repository contains a step-by-step guide on setting up a private, free AI assistant that runs locally on a Raspberry Pi 5 (tested on the 4GB model) using Ollama. You can then interact with this AI from the terminal of another Linux computer on your network using a simple bash function.

This is ideal for coding assistance, command explanations, and generating text without relying on cloud-based services or incurring API costs.

**The full, detailed guide with all setup commands is hosted on GitHub Pages:**

**➡️ [View the Full Guide Here](https://Aeell.github.io/askpi-local-ai-raspi5/)**

*(The guide content can also be found directly in the `index.md` file in this repository.)*

---

## Summary of Setup

The guide covers:
1.  **Raspberry Pi (Server):** Installing and configuring [Ollama](https://ollama.com/) for network access.
2.  **LLM:** Downloading and running a suitable lightweight model ([`gemma:2b`](https://ollama.com/library/gemma) recommended for 4GB RAM).
3.  **Linux Client (Computer):** Installing necessary tools ([`curl`](https://curl.se/), [`jq`](https://jqlang.github.io/jq/)) and creating a bash function (`askpi`) for easy prompting.

## Prerequisites Overview

* **Server Hardware:** Raspberry Pi 5 (4GB+ RAM)
* **Server OS:** Raspberry Pi OS (64-bit) or similar Linux
* **Client Hardware:** Any Linux Computer (Desktop/Laptop)
* **Client OS:** Linux distribution (Debian/Ubuntu/Fedora/Arch etc.) with Bash or Zsh
* **Network:** Both devices on the same local network

## Example Usage (Final Result)

Once configured following the full guide, you can use the `askpi` command on your client computer:

```bash
askpi "Write a python function to reverse a string"
askpi "Explain how SSH port forwarding works"

Notes
This guide uses a direct curl-based approach for the client-server interaction. While other tools like mods or sgpt exist, they presented configuration challenges during the testing phase for this specific setup, making the curl method more reliable in this context. The full guide provides more details.
