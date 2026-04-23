# 🦞 OpenClaw 🦞 AI Setup & Hardening Guide for Apple Silicon 🖥️
🚧 🚧 🚧 Under Construction 🚧 🚧 🚧
This document outlines the step-by-step guide for setting up a dedicated, secure, and highly instrumented development environment optimized for AI tooling.

## 1. 🛠️ Hardware & OS Preparation
Before installing AI tools, the host OS must be Updated, Hardened, and Configured for high-availability.

### System Settings
- **Identity:** Set up an administrator account via the macOS UI.
  - Pin **Terminal** to the dock.
  - Disable rotating MAC if using DHCP lease
  - Set Screen timeout (Settings > Lock Screen)
  - Settings > Energy >
    -- prevent automatic sleeping when the display is off
    -- Start up automatically after a power failure

- **Networking:** 
  - Set a **Static IP** for the machine.
  - Disable **Rotating MAC Addresses** if using a DHCP lease.
  - Set additional Wi-Fi networks to **Manual Join** only.

- **Display & Security:**
  - **Screen Timeout:** Set via *Settings > Lock Screen*.
  - **OS Updates:** Run all pending updates and reboot.

- **Power Management:** - *Settings > Energy*: 
    - Enable **Prevent automatic sleeping when the display is off**.
    - Enable **Start up automatically after a power failure**.

- **Remote Access:** - Enable **SSH** in Sharing.
  - Grant **Full Disk Access** to Terminal/SSH.
  - Setup SSH key pairs for each user account.

### User Configuration
- Create a secondary **User Account** with Admin privileges.
  - *Note: Admin rights will be removed after tool installation for security.*
- Log into the new account; skip Apple ID sign-in for security/privacy.
- Change the **Desktop Background** to a distinct color/image (visual cue for AI-specific environment).

---

## 2. 📦 Core Toolbelt Installation
Standardized environment setup using Homebrew and Oh My Zsh.

### Shell & Package Managers
```bash
# Install Oh My Zsh
sh -c "$(curl -fsSL [https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh](https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh))"

# Install Homebrew
/bin/bash -c "$(curl -fsSL [https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh](https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh))"

# Core Utilities
brew install git node htop tmux
```
