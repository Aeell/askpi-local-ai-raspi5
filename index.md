Okay, let's enhance the guide for a better GitHub Pages experience by adding links to official resources and ensuring the formatting is clean. GitHub Pages automatically renders Markdown files (index.md or README.md) into websites, and fenced code blocks usually get a copy button.

Here is the enhanced guide:

Markdown

# Local AI Assistant in Terminal using Raspberry Pi 5 (4GB) and Ollama

*Guide Version: 1.0 (Based on testing April 2025)*

## Introduction

Welcome! This guide details how to set up a private, free AI assistant accessible directly from your Linux terminal. We will use a [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/) (specifically tested on a **4GB RAM model**) to host a local Large Language Model (LLM) via [Ollama](https://ollama.com/), and interact with it from a Linux computer (e.g., running [Pop!_OS](https://pop.system76.com/), [Ubuntu](https://ubuntu.com/), [Fedora](https://fedoraproject.org/), [Arch Linux](https://archlinux.org/)) using a simple bash function.

The result is a convenient command (`askpi`) you can run on your client computer to get coding help, explanations, or boilerplate code generated locally, without relying on cloud services.

* **Benefit:** Private, free, offline-capable AI assistance.
* **Why this method?** While tools like [`mods`](https://github.com/charmbracelet/mods) or [`sgpt`](https://github.com/TheR1D/shell_gpt) exist, significant configuration challenges were encountered during testing on this specific setup. This guide uses a more direct [`curl`](https://curl.se/)-based approach which proved reliable for interacting with the Ollama API running on the Raspberry Pi.

---

## Prerequisites and Tools Needed

Before you start, ensure you have the required hardware and software.

**1. Raspberry Pi (Server):**

* **Hardware:**
    * [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/) (**4GB RAM Model Tested** - 8GB Recommended for more model flexibility)
    * MicroSD Card (16GB+, 32GB+ recommended)
    * Appropriate Power Supply for Pi 5
    * Network Connection (Ethernet cable strongly recommended for stability)
* **Operating System:**
    * [Raspberry Pi OS](https://www.raspberrypi.com/software/) (64-bit recommended) or compatible Linux distro (like Debian 12 Bookworm).
* **Software (to be installed or usually pre-installed):**
    * [`curl`](https://curl.se/) (for downloading Ollama)
    * `ssh` server ([OpenSSH](https://www.openssh.com/) likely, enabled via `sudo raspi-config` or equivalent if not default)
    * [`ufw`](https://help.ubuntu.com/community/UFW) (Optional, if you use the UFW firewall)

**2. Linux Computer (Client):**

* **Hardware:**
    * Any modern computer (desktop or laptop).
* **Operating System:**
    * Linux Distribution (Instructions provided for Debian/Ubuntu/Pop!_OS, Fedora, Arch Linux, openSUSE). Uses [Bash](https://www.gnu.org/software/bash/manual/) or [Zsh](https://www.zsh.org/) shell.
* **Software (to be installed or usually pre-installed):**
    * [`curl`](https://curl.se/) (for making API requests)
    * [`jq`](https://jqlang.github.io/jq/) (for parsing JSON responses from Ollama)
    * [`nano`](https://www.nano-editor.org/) (or your preferred terminal text editor like [`vim`](https://www.vim.org/), [`micro`](https://micro-editor.github.io/))
    * `ssh` client ([OpenSSH](https://www.openssh.com/) likely)

**3. Large Language Model (LLM):**

* **Model:** `gemma:2b`
    * **Source:** Google ([https://ai.google.dev/gemma](https://ai.google.dev/gemma))
    * **Ollama Hub:** [https://ollama.com/library/gemma](https://ollama.com/library/gemma) (We'll use the `gemma:2b` tag)
    * **Reason:** This 2-billion parameter model performs well and reliably fits within the 4GB RAM constraints of the tested Raspberry Pi 5. Larger models like Phi-3 Mini were found to cause memory issues during testing on this hardware.

---

## Phase 1: Raspberry Pi Setup (Ollama Server)

Configure your Raspberry Pi to host the Ollama service and the LLM.

1.  **Connect & Update:**
    * Connect to your Pi using SSH from your client computer's terminal (replace placeholders):
        ```bash
        ssh your_pi_username@YOUR_PI_IP_ADDRESS
        ```
    * *(If you don't know the IP, check your router's client list or use `ip addr show` on the Pi if using a direct monitor/keyboard. See [`ip` command docs](https://man7.org/linux/man-pages/man8/ip.8.html)).*
    * *(If you encounter SSH host key errors, follow the instructions given by the `ssh` command, often `ssh-keygen -R "YOUR_PI_IP_ADDRESS"` on the client).*
    * Update the Pi's operating system (run *on the Pi*):
        ```bash
        sudo apt update && sudo apt full-upgrade -y
        ```

2.  **Install Ollama:** (Run *on the Pi*)
    Download and run the Ollama installation script:
    ```bash
    curl -fsSL [https://ollama.com/install.sh](https://ollama.com/install.sh) | sh
    ```

3.  **Configure Ollama for Network Access:** (Run *on the Pi*)
    Allow Ollama to accept connections from your client computer. Edit the [systemd](https://systemd.io/) service configuration using `systemctl edit`:
    ```bash
    sudo systemctl edit ollama.service
    ```
    * In the editor (likely `nano`), add these lines exactly:
        ```ini
        [Service]
        Environment="OLLAMA_HOST=0.0.0.0"
        ```
    * Save and exit (`Ctrl+X`, then `Y`, then `Enter`).
    * Apply the changes by reloading the systemd daemon and restarting Ollama:
        ```bash
        sudo systemctl daemon-reload
        sudo systemctl restart ollama.service
        ```

4.  **Configure Firewall (If Active):** (Run *on the Pi*)
    If you use [`ufw`](https://help.ubuntu.com/community/UFW) firewall on the Pi, allow Ollama's default port (11434):
    ```bash
    # Check status first (optional)
    # sudo ufw status
    # Allow Ollama port if ufw is active
    sudo ufw allow 11434/tcp
    # Reload firewall if needed
    # sudo ufw reload
    ```

5.  **Pull the LLM:** (Run *on the Pi*)
    Download the [`gemma:2b`](https://ollama.com/library/gemma) model (recommended for 4GB RAM):
    ```bash
    ollama pull gemma:2b
    ```
    *(Optional: If you previously downloaded larger models, remove them to save space: `ollama rm model-name:tag`)*

6.  **Test Ollama Locally:** (Run *on the Pi*)
    Verify the model loads and responds:
    ```bash
    ollama run gemma:2b "Hi! Are you working?"
    ```
    You should get a response. Type `/bye` to exit the Ollama prompt.

7.  **Get Pi's IP Address:** (Run *on the Pi*)
    Confirm the IP address you'll need for the client setup:
    ```bash
    ip addr show | grep "inet "
    ```
    Note the IP address (e.g., `192.168.1.101`) associated with your `eth0` or `wlan0` interface.

8.  **Log Out:** You're done configuring the Pi server.
    ```bash
    exit
    ```

---

## Phase 2: Linux Computer Setup (Client)

Configure your main Linux computer to easily send prompts to the Pi.

**1. Install Client Tools:** (Run *on the Client Computer*)

Ensure you have the necessary command-line tools installed: [`curl`](https://curl.se/), [`jq`](https://jqlang.github.io/jq/), a text editor ([`nano`](https://www.nano-editor.org/) used in examples), and an `ssh` client (usually pre-installed).

Choose the command appropriate for your distribution's package manager:

* **For Debian / Ubuntu / Pop!\_OS / Mint (using [`apt`](https://wiki.debian.org/Apt)):**
    ```bash
    sudo apt update && sudo apt install curl jq nano -y
    ```

* **For Fedora (using [`dnf`](https://dnf.readthedocs.io/en/latest/)):**
    ```bash
    sudo dnf install curl jq nano -y
    ```

* **For Arch Linux / Manjaro (using [`pacman`](https://wiki.archlinux.org/title/pacman)):**
    ```bash
    sudo pacman -Syu --needed curl jq nano --noconfirm
    ```
    *(Note: `--needed` prevents reinstalling already up-to-date packages)*

* **For openSUSE (using [`zypper`](https://en.opensuse.org/Portal:Zypper)):**
    ```bash
    sudo zypper install curl jq nano
    ```

*(Note: If `nano` isn't your preferred editor, replace it with [`vim`](https://www.vim.org/), [`micro`](https://micro-editor.github.io/), etc., or skip installing it if you already have one.)*

**2. Create `askpi` Bash Function:** (Run *on the Client Computer*)

This step involves adding a helper function to your shell's configuration file. This function wraps the `curl` command for convenience.

* Open your shell configuration file. For the default [Bash](https://www.gnu.org/software/bash/manual/) shell, this is usually `~/.bashrc`. **If you use [Zsh](https://www.zsh.org/), edit `~/.zshrc` instead.** The function itself will work in either shell.
    ```bash
    # For Bash (most common)
    nano ~/.bashrc

    # If using Zsh
    # nano ~/.zshrc
    ```
* Scroll towards the end of the file. Find a suitable spot (e.g., after aliases, before other PATH exports) and paste the following function definition.
* **CRITICAL:** Replace `YOUR_PI_IP_ADDRESS` inside the function with the actual IP address of *your* Raspberry Pi.

    ```bash
    # --- Custom Function for Local Ollama on Pi ---
    # Usage: askpi "Your prompt here"
    askpi() {
      # --- CONFIGURATION ---
      # !!! IMPORTANT: Replace with your Pi's actual IP address !!!
      local pi_ollama_ip="YOUR_PI_IP_ADDRESS"
      local ollama_port="11434"
      # Ensure this matches the model pulled on the Pi (e.g., gemma:2b)
      local target_model="gemma:2b"
      # --- END CONFIGURATION ---

      # Check if jq is installed
      if ! command -v jq &> /dev/null; then
        echo "Error: jq is not installed. Please install it first." >&2
        return 1
      fi

      # Check if prompt is provided
      if [ -z "$@" ]; then
        echo "Usage: askpi \"Your prompt here\"" >&2
        return 1
      fi

      # Escape quotes within the prompt for JSON compatibility
      local prompt_content
      prompt_content=$(echo "$@" | sed 's/"/\\"/g')

      # Use curl -s for silent operation (remove -s to see progress/errors)
      # Use --connect-timeout 15 for a 15-second connection timeout
      # Use --max-time 300 for a 5-minute maximum total time
      curl -s --connect-timeout 15 --max-time 300 "http://${pi_ollama_ip}:${ollama_port}/api/chat" -d "{
        \"model\": \"${target_model}\",
        \"messages\": [ { \"role\": \"user\", \"content\": \"$prompt_content\" } ],
        \"stream\": false
      }" | jq -r '.message.content'
    }
    # --- End Custom Function ---
    ```
* Save and exit (`Ctrl+X`, `Y`, `Enter` in nano).

**3. Apply Shell Configuration Changes:** (Run *on the Client Computer*)
Load the new function into your current shell session:
    ```bash
    # For Bash
    source ~/.bashrc

    # If using Zsh
    # source ~/.zshrc
    ```
*(Note: The function will be available automatically in new terminal windows you open).*

---

## Phase 3: Usage

You can now use the `askpi` command directly in your client computer's terminal to interact with the `gemma:2b` model running on your Raspberry Pi.

1.  **Basic Usage:**
    Run the command followed by your prompt enclosed in quotes:
    ```bash
    askpi "YOUR PROMPT HERE"
    ```

2.  **Examples & Expected Output:**

    * **Example 1: Get code**
        Command:
        ```bash
        askpi "Write a python hello world example"
        ```
        Expected Output (will vary slightly in wording/formatting):
        ```text
        ```python
        print("Hello, world!")
        ```

        This code prints the classic "Hello, world!" message to the console when run with Python.
        ```

    * **Example 2: Explain command**
        Command:
        ```bash
        askpi "Explain the bash command ls -l"
        ```
        Expected Output (will vary slightly, potentially more/less verbose):
        ```text
        The command `ls -l` lists files and directories in the current location using the "long" format. This provides detailed information for each item, typically including:
        - File type and permissions (e.g., `-rw-r--r--`, `drwxr-xr-x`)
        - Number of hard links
        - Owner username
        - Group name
        - File size in bytes
        - Last modification date and time
        - File or directory name
        ```

    * **Example 3: Generate Boilerplate**
        Command:
        ```bash
        askpi "Generate a basic HTML5 document structure"
        ```
        Expected Output (will vary slightly):
        ```html
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>Document</title>
        </head>
        <body>

        </body>
        </html>
        ```

**Important Note:** Run `askpi` with one distinct prompt at a time. Providing multiple `askpi` commands or arguments intended as separate prompts on the same line (like `askpi prompt1 askpi prompt2`) will likely result in everything after the first `askpi` being treated as part of a single, combined prompt to the LLM.

---

## Troubleshooting & Notes

* **Finding Pi IP:** If your Pi's IP address changes, update the `pi_ollama_ip` variable inside the `askpi` function in your shell config file (`~/.bashrc` or `~/.zshrc`), then run `source ~/.bashrc` (or `source ~/.zshrc`) again. Consider setting a static IP or DHCP reservation for the Pi in your router settings.
* **Check Ollama Service:** If `askpi` fails, SSH into the Pi and check the [systemd](https://systemd.io/) service using [`systemctl`](https://www.freedesktop.org/software/systemd/man/systemctl.html):
    ```bash
    sudo systemctl status ollama.service
    ```
    Restart it if needed:
    ```bash
    sudo systemctl restart ollama.service
    ```
* **RAM Limits:** The 4GB Pi is limited. If `gemma:2b` causes "could not load model" errors (check Ollama logs on Pi), try `orca-mini`. Pull it (`ollama pull orca-mini`) and update the `target_model` in the `askpi` function.
* **`jq` Error:** If `askpi` gives raw JSON or `jq` errors, ensure [`jq`](https://jqlang.github.io/jq/) is installed on your client computer (use the relevant command from Phase 2, Step 1).
* **Network Issues:** Use [`ping`](https://man7.org/linux/man-pages/man8/ping.8.html) `YOUR_PI_IP_ADDRESS` from the client computer. Check firewalls on both devices (port 11434 needs to be allowed *incoming* on the Pi).
* **Curl Timeouts:** Adjust `--connect-timeout` and `--max-time` in the `askpi` function if needed, but long waits often indicate Pi-side issues.
* **Client Tool Alternatives:** This guide uses `curl` due to configuration issues encountered with [`mods`](https://github.com/charmbracelet/mods) and [`sgpt`](https://github.com/TheR1D/shell_gpt) in testing. Those tools might work with different versions or further debugging, but this method proved most reliable for this specific Pi 5 (4GB) + Ollama setup.

---
