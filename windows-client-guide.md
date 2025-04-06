# Connecting to Local Ollama/Pi AI Assistant from Windows

This guide explains how Windows users can connect to the Ollama LLM server running on the Raspberry Pi, as set up in the [main guide](index.md).

## Method 1: Using WSL (Windows Subsystem for Linux) - Recommended

The easiest and most consistent way to use the `askpi` function from Windows is via WSL. This lets you run a Linux distribution directly on Windows and follow the exact same client setup steps as native Linux users.

**Prerequisites:**

1.  **Windows:** Windows 10 (version 2004 or higher) or Windows 11.
2.  **WSL Installed:** You need WSL enabled and a Linux distribution installed. Ubuntu is a common choice available from the Microsoft Store.
    * **Official WSL Installation Guide:** [https://learn.microsoft.com/en-us/windows/wsl/install](https://learn.microsoft.com/en-us/windows/wsl/install)
3.  **Ollama Pi Server:** Your Raspberry Pi must be set up and running Ollama according to Phase 1 of the [main guide](index.md).
4.  **Network:** Your Windows computer and Raspberry Pi must be on the same local network. *(See WSL Networking note below).*

**Setup Steps (Inside WSL):**

1.  **Open WSL Terminal:** Launch your installed Linux distribution (e.g., Ubuntu) from the Windows Start Menu or Windows Terminal. You will now be in a Linux shell environment.
2.  **Follow Main Guide's Phase 2:** Execute the steps exactly as described in "[Phase 2: Linux Computer Setup (Client)](index.md#phase-2-linux-computer-setup-client)" of the main guide, but perform them *inside your WSL terminal*. This includes:
    * **Installing `curl` and `jq`:**
        ```bash
        # Inside your WSL terminal (e.g., Ubuntu)
        sudo apt update && sudo apt install curl jq nano -y
        ```
    * **Creating the `askpi` Bash Function:** Edit your WSL Linux environment's `~/.bashrc` (or `~/.zshrc` if using Zsh in WSL) using `nano` and paste the `askpi` function code from the main guide, making sure to replace `YOUR_PI_IP_ADDRESS` with your Pi's actual IP.
        ```bash
        # Inside your WSL terminal
        nano ~/.bashrc
        # (Paste the function definition from index.md)
        # Save and exit (Ctrl+X, Y, Enter)
        ```
    * **Applying `.bashrc` Changes:**
        ```bash
        # Inside your WSL terminal
        source ~/.bashrc
        ```

3.  **Usage (Inside WSL):**
    You can now use the `askpi` command directly within your WSL terminal, just like on a native Linux machine:
    ```bash
    # Inside your WSL terminal
    askpi "Write a PowerShell script to list running services"
    ```
    ```bash
    askpi "Explain the difference between CMD and PowerShell"
    ```

**WSL Networking Note:** Usually, WSL can directly access devices on your local network (like your Pi). If you encounter connection issues (e.g., `curl` or `ping` from WSL cannot reach the Pi's IP address), you may need to investigate WSL's network settings. Sometimes restarting WSL or checking Windows firewall rules related to WSL might be necessary. Refer to Microsoft's WSL networking documentation for advanced troubleshooting: [https://learn.microsoft.com/en-us/windows/wsl/networking](https://learn.microsoft.com/en-us/windows/wsl/networking)

---

## Method 2: Native Windows (PowerShell - Advanced)

This method avoids WSL but requires using native Windows tools and PowerShell scripting. It's more complex to set up.

**Prerequisites:**

1.  **Windows PowerShell:** Included in modern Windows.
2.  **`curl.exe`:** Usually included in Windows 10/11 and available in the default PATH. Test with `curl --version` in PowerShell.
3.  **`jq.exe`:** Needs to be downloaded manually and added to your PATH.
    * Go to [https://jqlang.github.io/jq/download/](https://jqlang.github.io/jq/download/).
    * Download the 64-bit Windows executable (`jq-win64.exe`).
    * Rename it to `jq.exe`.
    * Place `jq.exe` in a folder included in your Windows System PATH environment variable (e.g., create `C:\Tools` and add `C:\Tools` to your System's PATH variable via "Edit the system environment variables" settings). A restart or re-login might be needed. Test with `jq --version` in PowerShell.
4.  **Ollama Pi Server:** Set up as per Phase 1 of the [main guide](index.md).
5.  **Network:** Windows computer and Pi on the same network.

**Setup Steps:**

1.  **Create/Edit PowerShell Profile:** Your PowerShell profile script runs automatically when PowerShell starts.
    * Check if it exists: In PowerShell, run `Test-Path $PROFILE`
    * If `False`, create it: `New-Item -Path $PROFILE -Type File -Force`
    * Open it for editing: `notepad $PROFILE`

2.  **Add PowerShell Function:** Paste the following PowerShell function code into the Notepad window:

    ```powershell
    # --- Custom Function for Local Ollama on Pi (PowerShell) ---
    # Usage: AskPi "Your prompt here"
    function AskPi {
        param(
            [Parameter(Mandatory=$true, Position=0, ValueFromRemainingArguments=$true)]
            [string[]]$PromptArray
        )

        # --- CONFIGURATION ---
        # !!! IMPORTANT: Replace with your Pi's actual IP address !!!
        $PiOllamaIp = "YOUR_PI_IP_ADDRESS"
        $OllamaPort = "11434"
        # Ensure this matches the model pulled on the Pi
        $TargetModel = "gemma:2b"
        # --- End Configuration ---

        $Prompt = $PromptArray -join ' ' # Join arguments into a single string prompt
        $Uri = "http://${PiOllamaIp}:${OllamaPort}/api/chat"

        # Construct the JSON body payload
        $Body = @{
            model = $TargetModel
            messages = @(
                @{
                    role = "user"
                    content = $Prompt
                }
            )
            stream = $false
        } | ConvertTo-Json -Depth 5 # Use -Depth 5 for nested objects

        try {
            # Make the API request using curl.exe
            # We pipe the JSON body to curl's stdin and parse curl's output with jq
            # Use Write-Host for intermediate checks if needed, use Write-Output for final result
            $jsonResponse = $Body | curl.exe -s --connect-timeout 15 --max-time 300 -X POST -H "Content-Type: application/json" --data @- $Uri

            if ($LASTEXITCODE -ne 0) {
                 Write-Error "curl.exe failed with exit code $LASTEXITCODE"
                 # Optionally write $jsonResponse here if it contains error details from curl itself
                 return
            }

            # Parse the JSON response with jq and extract the content
            $content = $jsonResponse | jq.exe -r '.message.content'

            if ($LASTEXITCODE -ne 0) {
                 Write-Error "jq.exe failed to parse response. Raw response: $jsonResponse"
                 return
            }

            # Output the assistant's message content
            Write-Output $content

        } catch {
            Write-Error "Error processing Ollama request on Pi: $($_.Exception.Message)"
        }
    }
    # --- End Custom Function ---
    ```
    * **CRITICAL:** Replace `YOUR_PI_IP_ADDRESS` with your Pi's actual IP address.

3.  **Save and Close** Notepad.

4.  **Reload Profile:** Open a **new** PowerShell window. The `AskPi` function should now be available. (Alternatively, in an existing window, run `. $PROFILE`). *Note: You might need to adjust PowerShell's execution policy if profile scripts don't run (`Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`).*

**Usage (From PowerShell):**

```powershell
AskPi "Write a PowerShell command to get the current user"
AskPi "Explain the difference between Invoke-WebRequest and Invoke-RestMethod"
Disclaimer: This PowerShell method relies on external curl.exe and jq.exe being correctly installed and in the PATH.
PowerShell scripting and environment variable handling differ significantly from Linux shells.
The WSL method is generally recommended for simplicity and consistency with the main guide.

Conclusion
For Windows users, using WSL is the most direct way to leverage the Linux-based setup and the askpi bash function detailed in the main guide.
The native PowerShell method provides an alternative but requires more manual setup and familiarity with PowerShell. Choose the method that best suits your comfort level!
