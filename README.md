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

### Install Oh My Zsh - Shell Extension 
```sh
# Install Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### Install Homebrew Package Manager 
```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Install Brew Utilities
```zsh
brew install git node htop tmux
brew install ollama
brew services start ollama
brew install --cask obsidian
brew install asitop
brew install repomix
```

### Install Stats to Monitor System Temps
```zsh
mkdir ~/Applications && brew install --cask --appdir=~/Applications stats
```

### Install Mac Fan Control 
```zsh
brew install --cask macs-fan-control
```

### Install 'uv' and fluidtop 
```zsh 
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env sh, bash, zsh
sudo uvx fluidtop
```
