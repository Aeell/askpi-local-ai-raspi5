# Local AI Assistant in Terminal using Raspberry Pi 5 (4GB) and Ollama

This repository contains a step-by-step guide on setting up a private, free AI assistant that runs locally on a Raspberry Pi 5 (tested on the 4GB model) using Ollama. You can then interact with this AI from the terminal of another Linux computer on your network using a simple bash function.

This is ideal for coding assistance, command explanations, and generating text without relying on cloud-based services or incurring API costs.

**The full, detailed guide is hosted on GitHub Pages:**

**➡️ [View the Full Guide Here](https://Aeell.github.io/askpi-local-ai-raspi5/)**
*(Replace `<your-username>` and `<your-repository-name>` with your actual GitHub username and repository name after deploying)*

*(The main guide content can also be found directly in the `index.md` file in this repository.)*

**Windows Users:** Instructions for connecting from a Windows computer (using WSL or native tools) are available in the [Windows Client Guide](windows-client-guide.md).

---

## Summary of Setup

The guide covers:
1.  **Raspberry Pi (Server):** Installing and configuring [Ollama](https://ollama.com/) for network access.
2.  **LLM:** Downloading and running a suitable lightweight model ([`gemma:2b`](https://ollama.com/library/gemma) recommended for 4GB RAM).
3.  **Linux Client (Computer):** Installing necessary tools ([`curl`](https://curl.se/), [`jq`](https://jqlang.github.io/jq/)) and creating a bash function (`askpi`) for easy prompting.

## Prerequisites Overview

* **Server:** Raspberry Pi 5 (4GB+ RAM), Raspberry Pi OS (64-bit recommended), Ollama.
* **Client:** Linux Computer (Desktop/Laptop) or Windows (using WSL recommended, see Windows guide), Bash/Zsh Shell (for Linux/WSL), `curl`, `jq`.
* **LLM:** A model suitable for the Pi's RAM (Guide uses `gemma:2b`).
* **Network:** Both devices on the same local network.

## Example Usage (Final Result)

Once set up following the full guide (or the Windows guide), you can use the `askpi` command (or `AskPi` in PowerShell) on your client computer:

```bash
# Linux / WSL Example
askpi "Write a python function to check if a number is prime"
PowerShell

# Native Windows PowerShell Example (if using that method)
AskPi "Write a PowerShell script to list files modified in the last day"
Notes
This guide primarily uses a curl-based approach for the client because significant configuration issues were encountered with alternative tools like mods and sgpt during testing on this specific hardware/software combination. The curl method proved to be the most reliable way to interact directly with the Ollama API in this scenario.
