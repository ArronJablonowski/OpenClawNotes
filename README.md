# 🦞 OpenClaw 🦞 AI Lab Setup & Hardening Guide for Apple Silicon 🖥️

🚧 🚧 🚧 Under Construction 🚧 🚧 🚧

This document outlines a step-by-step guide for setting up a dedicated, *mostly* secure, OpenClaw installation locally on Apple M series systems. 

# The Security Risk tldr; 
1. If the OpenClaw system becomes compromised, any data on that system should also be considered compromised. Files, passwords, api keys, etc. *ie. don't store sensitive data on the OpenClaw system*
2. If the OpenClaw system becomes compromised and it has access to anything else on the local network, that compromise could potentially spread across the local network to additional devices and data stores.
### ⚠️ Yes, the OpenClaw system can still become compromised by things outside of your conrol. Especially since OpenClaw uses npm packages, and supply chain attacks could unknowing occure and infect your system through compromised package updates. 
  - npm supply chain attacks have been in the Cyber Security news lately. Examples:
  - one ..
  - two .. 

# 🧪 My OpenClaw setup recommendations for securing an AI Lab Environment 

## 1. 📡 Network Isolation 
Before proceeding it is important to isolate the OpenClaw system from the rest of the LAN (local area network). *see The Security Risks above*

### Put the sytem on its own dedicated and isolated VLAN. *VLANs are outside the scope of this guide - google VLAN* 

## 2. 👷‍♂️ Admin User Configuration 
- **Make an Admin User:** This account will be used to run admin tasks only, and NOT LLMs. 
  - Set a static (fixed) IP address.    
  - Disable rotating MAC if using a DHCP lease and WiFi *(Settings > Network > WiFi > SSID's (...) > Network Settings > Private WiFi address > Set to: "Fixed")*
  - Set Screen timeout *(Settings > Lock Screen)*
  - Set Energy settings *(Settings > Energy: -- turn low power mode off -- prevent automatic sleeping when the display is off should be turned on -- Start up automatically after a power failure should be turned on)*

## 3. 🛰️ Allow Remote Access
- **Enable SSH** *(Settings > General > Sharing > Remote Login: toggle on)*
  - Grant **Full Disk Access** to Terminal/SSH.
  - Setup SSH key pairs for the admin user.

## 4. 🤖 LLM / AI Agent User Configuration
- **Create a secondary User Account** with (temporary) admin privileges. 
  - *Note: Admin rights will be removed after tool installation for security.*
- Now log into the new secondary account; skip Apple ID sign-in for security/privacy.
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
- *Open Mac Fac Control in the GUI. {config}*

### 4f. Install 'uv' and fluidtop to Monitor Performance
```zsh 
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env sh, bash, zsh
sudo uvx fluidtop
```

### 4g. Modify the Terminal's Prompt 
This will be a visual que to help identify when you're in the LLM/AI user's terminal, and help prevent entering commands into the wrong terminal window. 
```
nano ~/.zshrc
```
- Add the following to the end of the .zshrc file:
```
# This function prepends the 'Brain' emoji (🧠) to the prompt for security awareness.
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
Add commands to the sudoer's file so your secondary LLM/AI user will be allowed to run some commands as root. 
- vi commands: 
  - press 'i'              *enter insert mode*
  - press '{esc} : q!'     *quit vi, without writing changes to the file*
  - press '{esc} : wq'     *write the changes, and then quit vi*
  - press '{esc} : w'      *Save (write) the file but stay in the editor*
  - press 'ZZ'             *while in Command Mode, no colon needed - Save and quit q*
```
sudo visudo
```
- Add the following to the end of the file, and replace 'lobster_user' with your LLM/AI user's name: 
```
lobster_user ALL=(ALL) NOPASSWD: /usr/bin/powermetrics
lobster_user ALL=(ALL) NOPASSWD: /opt/homebrew/bin/asitop
lobster_user ALL=(ALL) NOPASSWD: /Users/aj_lab/.local/bin/uvx fluidtop
```

### 4k. Install 🦞 OpenClaw 🦞 
```
curl -fsSL https://openclaw.ai/install.sh | bash
```
As you go through the installer script: 
- keep gemma4 as default
- Enable 'command-logger' & 'session-memory'
