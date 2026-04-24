# 🦞 OpenClaw 🦞 AI Lab Setup & Hardening Guide for Apple Silicon 

🚧 🚧 🚧 Under Construction 🚧 🚧 🚧
*in beta* 

This document provides a guide for setting up a dedicated, locally hosted, and security-hardened installation of OpenClaw on Apple Silicon hardware. 

 🎯 The Objective - I developed this guide for two primary reasons:
- First, to personally explore OpenClaw as an AI Agent, and experiment with it's security, given its reputation as a "high-risk" autonomous AI agent.  
- Second, during my research into OpenClaw, I noted the prevalence of poor and generic security advice lacking depth. Consequently, I aimed to create a hands-on hardening guide that promotes higher security standards and implements security best practices to address common vulnerabilities and conceptual misunderstandings related to security.

##

# 🚨 The Security Risks 🚨
***🛑 PRIMARY WARNING:*** If the OpenClaw system is compromised, **all data on that system should also be considered compromised.** This includes local files, passwords, API keys, Keychains, etc. **DO NOT store sensitive data on the OpenClaw system, and DO NOT use your daily driver Mac to run OpenClaw.**

### ⚠️ Supply Chain Vulnerabilities
The biggest immediate risk to your OpenClaw is the supply chain. Since OpenClaw relies on community developed `skills` and `npm` packages, a compromised `skill` or `npm` dependency could unknowingly infect your system through malicious package updates.

Don't let your environment auto-update itself into a compromise.
- Version Pinning: If you are using npm to install skill dependencies, use the --save-exact flag. This prevents npm install from pulling in a "minor update" that might actually be a malicious injection (like the Axios compromise of early 2026).
- Manual Audits: Before running a new skill, run npm audit inside the skill's directory.
``` cd ~/.openclaw && npm audit && npm list --depth=0 ```

### ⚠️ Command Injection Vulnerabilities 
In OpenClaw, command injection isn't just a coding bug; it’s an architectural risk known as the Lethal Trifecta. 
 - **Access to Untrusted Content:** The agent can read emails, browse the web, or monitor messaging apps.
 - **Autonomous Tool Use:** The agent has a "Skill" to run shell commands (bash, zsh, terminal).
 - **Persistence:** The agent maintains a long-term memory and can run commands and crons (scheduled tasks) in the background.

**🔑 SECRETS WARNING:** OpenClaw stores API tokens and session keys in plaintext within `~/.openclaw/credentials/`. Never include these in your Git commits or share your `SOUL.md` file if it contains sensitive environmental details.

**🕸️ Potential Spread:** Without proper network segmentation, a compromised OpenClaw instance may be leveraged as a pivot point for unauthorized access to other internal network devices and network-attached storage.

# 🧪 My OpenClaw Setup Recommendations for Securing a Locally Hosted AI Lab Environment 🧪

## 1. 📡 Network Segmentation & Isolation - 💣 *The Blast Radius Control*  💣
Before proceeding it is important to isolate the OpenClaw system from the rest of the LAN (local area network). *See The Security Risks above*

### 🛡️ Put the OpenClaw system on its own dedicated and isolated VLAN. 
By enforcing strict network segmentation, this configuration mitigates the threat of internal pivoting and contains potential adversarial activity within the isolated network segment. 
- *VLANs are outside the scope of this guide - google 'what is a VLAN' - or ask a cloud AI provider* 
- *Isolation via Docker is not recommended as docker will cause a performance hit to the LLMs. (~10-20% slower)*

## 2. 🧑‍💻 Admin User Configuration 
If the system has been used for personal use, a brand new admin account should be created. Then delete out any prior personal use accounts from the system entirely. *Best practice would be to fully re-image/reload MacOS for a fresh system.* 

- **Make an Admin User:** This account will be used to run admin tasks only, and NOT LLMs.
  - Set a static (fixed) IP address.    
  - Disable rotating MAC if using a DHCP lease and WiFi *(Settings > Network > WiFi > SSID's (...) > Network Settings > Private WiFi address > Set to: "Fixed")*
  - Set Screen timeout *(Settings > Lock Screen)*
  - Set Energy settings *(Settings > Energy: -- turn low power mode off -- prevent automatic sleeping when the display is off should be turned on -- Start up automatically after a power failure should be turned on)*

## 3. 🛰️ Allow Remote Access
- **Enable SSH** *(Settings > General > Sharing > Remote Login: toggle on)*
  - Grant **Full Disk Access** to Terminal/SSH.
  - Setup SSH key pairs for the admin user.

## 4. 🤖 LLM / AI User Configuration
- **Create a secondary User Account** with (*temporary*) admin privileges *(Settings > Users & Groups > Add User)*
  - *Note: Admin rights will be removed after tool installation for security / principle of least privilege.*
- Now log into the new **secondary account** via the GUI; skip Apple ID sign-in for security and privacy.
  - *Note: The following installs and commands will happen under this secondary account.*
- Change the **Desktop Background** to a distinct color/image as a visual cue for AI-specific environment.
- Pin the terminal to the dock.

### 4a. Install Oh My Zsh - Shell Extension 
```sh
# Install Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 4b. Install Homebrew - Package Manager 
```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 4c. Install Brew Utilities
```zsh
brew install git node htop tmux
brew install ollama
brew services start ollama
brew install --cask obsidian
brew install asitop
brew install repomix
```

### 4d. Install Stats to Monitor System Temps
```zsh
mkdir ~/Applications && brew install --cask --appdir=~/Applications stats
```
- *Open Stats in the GUI. {config}*

### 4e. Install Mac Fan Control 
```zsh
brew install --cask macs-fan-control
```
- *Open Mac Fan Control in the GUI. {config}*

### 4f. Install 'uv' and fluidtop to Monitor Performance
```zsh 
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env sh, bash, zsh
sudo uvx fluidtop
```

### 4g. Modify the Terminal's Prompt 
This will be a visual cue to help identify when you're in the LLM/AI user's terminal, and help prevent entering commands into the wrong terminal window. 
```
nano ~/.zshrc
```
- Add the following to the end of the .zshrc file:
```
# This function prepends the 'Brain' emoji (🧠) to the prompt for a visual cue/awareness.
preprompt_brain() {
    # \e[33m is the ANSI code for yellow/amber, making it noticeable.
    echo -e "\e[33m🧠\e[0m " 
}
# Hook the function into the Zsh prompt drawing routine.
precmd() {
    preprompt_brain
}
```
- Reload the zshrc file
```
source ~/.zshrc
```

### 4h. Harden ssh to only allow keypair auth 
Ensure you've already setup keypairs for both users. 
```
sudo nano /etc/ssh/sshd_config.d/00-disable-passwords.conf
```
- Add the following to the 00-disable-passwords.conf file: 
```
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
PermitEmptyPasswords no
```
### 4i. Modify tmux config for ez mode (like scrolling with your mouse or trackpad) 
```
nano ~/.tmux.conf
```
- Add the following to the .tmux file:
```
# Enable mouse support (clicking and scrolling)
set -g mouse on

# Start window and pane numbering at 1 (instead of 0)
set -g base-index 1
setw -g pane-base-index 1

# Don't rename windows automatically (keep your labels)
set-option -g allow-rename off
```

### 4j. sudo visudo  
- vi editor commands cheat sheet: 
  - press 'i'              *enter insert mode*
  - press '{esc} : q!'     *quit vi, without writing changes to the file*
  - press '{esc} : wq'     *write the changes, and then quit vi*
  - press '{esc} : w'      *Save (write) the file but stay in the editor*
  - press 'ZZ'             *while in Command Mode, no colon needed - Save and quit q*
    
Add commands to the sudoer's file so your secondary LLM/AI user will be allowed to run some commands as root.
```
sudo visudo
```
- Add the following to the end of the file, and replace *'lobster_user'* with your LLM/AI user's name: 
```
lobster_user ALL=(ALL) NOPASSWD: /usr/bin/powermetrics
lobster_user ALL=(ALL) NOPASSWD: /opt/homebrew/bin/asitop
lobster_user ALL=(ALL) NOPASSWD: /Users/lobster_user/.local/bin/uvx fluidtop
```

### 4k. Install OpenClaw 🦞 
```
curl -fsSL https://openclaw.ai/install.sh | bash
```
As you go through the installer script: 
- keep gemma4 as default
- Enable 'command-logger' & 'session-memory'

## 5. 🧑‍💻 Log out of your LLM/AI user and log into your Admin User again
- **Remove Admin/Sudo from the LLM/AI User:** Now that everything is installed and running properly under the LLM/AI user, we will revoke the user's admin/sudo access. 
  - Remove Admin (settings > Users & Groups > *click 'i' next to LLM/AI user's name* and toggle off the "allow this user to administer this computer")
  - Reboot the system so these changes take effect.
  - Verify the LLM/AI user no longer has Admin privileges. 

## 6. 🤖 Login to LLM/AI User Account & Tell OpenClaw to Run a Security Audit
```bash
openclaw security audit --deep
```

If you want to fix any issues that are found by the audit:
```bash
openclaw security audit --fix 
```

## 7. 🔒 Hardening the OpenClaw Agent (Zero-Trust Deployment)

You must treat an autonomous agent as **untrusted code execution**. By default, OpenClaw has the same permissions as your `LLM/AI` user. If the agent is "prompt injected" while reading a malicious file, email, or webpage, it could trick the system into running malicious commands and/or compromise the system. 

### **Implement "Least Privilege" Command Filtering**
OpenClaw’s `exec` tool is a powerful vector. Restrict it to a "Default Deny" posture by configuring an allow list of approved binaries.

1. Edit your tool policy: `nano ~/.openclaw/config.yaml`
2. Update the `tools.exec` section:
```yaml
tools:
  exec:
    security: "allowlist" # Changes from 'full' to 'allowlist'
    ask: "on-miss"       # Prompt you if it tries a command not on the list
    allow:
      - "git"
      - "repomix"
      - "ls"
      - "cat"
      - "python3"
    deny:
      - "brew"      # Never allow the agent to install software
      - "sudo"      # Absolute restriction 
      - "curl"      # Prevent direct exfiltration via CLI
      - "wget"      # Prevent direct exfiltration via CLI
```
Modify the allow and deny list as needed. Any binary not listed should prompt you before executing. 

## 8. 🧦 SOC-Grade Security Monitoring 
To achieve a stronger and more mature security posture, it is imperative to implement continuous telemetry monitoring of both host-level and network-layer activity. This dual-visibility approach remains the most effective methodology for identifying early Indicators of Compromise (IoCs) and mitigating potential security incidents.

- Host monitoring
  - Setup EDR and log forwarding agents, such as Elastic Agents, Wazuh, etc. 

- Network monitoring 
  - Setup port mirroring, and feed the mirrored network traffic into something like a Zeek/Bro sensor, or my personal favorite, a Security Onion sensor. 

## 📜 Quick Reference 
- Default OpenClaw Gateway: 127.0.0.1:18789 
- Default ollama base URL: 127.0.0.1:11434

Common OpenClaw commands:
- openclaw onboard
- openclaw security audit
- openclaw security audit --deep
- openclaw security audit --fix
- openclaw update status
- openclaw hooks list 
- openclaw hooks enable <name>
- openclaw hooks disable <name>
- openclaw --help
OpenClaw stores its files in a hidden folder ```~/.openclaw```

Common ollama commands: 
- ollama list
- ollama ps
- ollama pull <model_name>
- ollama run <model_name>
- ollama rm <model_name>
Ollama stores models in a hidden folder ```~/.ollama/models```

repomix: 
- cd into repo's root and run 'repomix' - this will output a file to that can be fed to AI models
```
# Example feeding repomix-output to an AI model for analysis
cat repomix-output.xml | ollama run qwen2.5-coder:7b "I have attached my repository in XML format. Please analyze the logic flow between the main entry script and the sub-modules."
````

OpenClaw Docs: 
- https://docs.openclaw.ai/gateway/remote
- https://docs.openclaw.ai/web/control-ui

OpenClaw FAQ: 
- https://docs.openclaw.ai/start/faq

OpenClaw: 
- https://docs.openclaw.ai/

ollama: 
- https://ollama.com/

OhMyZsh: 
- https://ohmyz.sh/

##

## AppleID Cleanup - *If you MUST use an existing user's account...*
If you must use a previously used personal account for some reason, here are some basic cleanup steps. 

- **⚠️ Log Out of Your AppleID and Cleanup Synced Data:** This will limit the impact if your system gets compromised, and the attack is able to elevate privileges. 
  - If you've ever logged into your AppleID using this account or any account on the system, log out. *(Settings > click on your AppleID > click Log Out)*
  - When promoted uncheck all the boxes to save cloud data locally. Delete all of your pictures, and personal data from your user's directories.
  - Check you Keychain & Passwords app, and delete out any stored credentials, etc. that are not needed for your lab environment. *(!! Ensure you're logged out of your AppleID first so you don't delete/modify your synced Keychain)* 
  - Delete any synced iMessages or texts using the commands below. Replace the "{admin_user}" with your admin user's account name. 
 ```
# Kill the background agents first so they don't lock the files
killall -9 IMDPersistenceAgent 2>/dev/null
killall -9 identityservicesd 2>/dev/null
killall -9 Messages 2>/dev/null

# Wipe the primary database and sync logic
rm -rf /Users/{admin_user}/Library/Messages/chat.db*
rm -rf /Users/{admin_user}/Library/Messages/Sync/*

# Wipe the attachments (often the largest risk for data exfiltration)
rm -rf /Users/{admin_user}/Library/Messages/Attachments/*

# Wipe the CloudKit sync caches and metadata
rm -rf /Users/{admin_user}/Library/Messages/CloudKit*
rm -rf /Users/{admin_user}/Library/Containers/com.apple.iChat/Data/Library/Caches/*

# Clear the knowledge database (which tracks who you talk to)
rm -rf /Users/{admin_user}/Library/Suggestions/com.apple.mobilephone/*
rm -rf /Users/{admin_user}/Library/Suggestions/com.apple.iChat/*
 ```
You may have to run these commands as root. If this still fails, then setup keypairs for the root user and login directly as the root user over ssh. 
```
# change to root user
sudo su

# cd to root's home, and pwd to ensure you are in root's home directory - /var/root
cd ~/ && pwd 

# make .ssh folder, cd into .ssh folder, make authorized_keys file
mkdir .ssh && cd .ssh && touch authorized_keys

# put your public key pair into the authorized_keys file
echo "{your_public_key}" >> authorized_keys
```
Now you should be able to ssh into the system as the root user. *Assuming you've already allowed Remote Access on this system - step 3*


*Note:* Don't forget to clean up other areas of your user profile - local files, browser files (browser history, cookies, etc.), and any other sensitive information you would want to keep private. 
