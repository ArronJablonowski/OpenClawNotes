# 🦞 OpenClaw 🦞 AI Lab Setup & Hardening Guide for Apple Silicon 

🚧 🚧 🚧 Under Construction 🚧 🚧 🚧
*in beta* 

This document provides a guide for setting up a dedicated, locally hosted, and security-hardened installation of OpenClaw on Apple Silicon hardware. 

 🎯 The Objective - I developed this guide for two primary reasons:
- First, to personally explore OpenClaw as an AI Agent, and experiment with it's security, given its reputation as a "high-risk", inherently difficult to secure autonomous AI agent.  
- Second, during my research into OpenClaw, I noted the prevalence of poor and generic security advice lacking depth. Consequently, I aimed to create a hands-on hardening guide that promotes higher security standards and implements security best practices to address common vulnerabilities and conceptual misunderstandings related to security.

##

# 🚨 The Security Risks 🚨
***🛑 PRIMARY WARNING:*** If the OpenClaw system is compromised, **all data on that system should also be considered compromised, unless you have evidence/logs that prove otherwise.** This includes local files, passwords, API keys, Keychains, etc. **DO NOT store sensitive data on the OpenClaw system, and DO NOT use your daily driver Mac to run OpenClaw.**

### ⚠️ Supply Chain Vulnerabilities
The biggest immediate risk to your OpenClaw is the supply chain. Since OpenClaw relies on community developed `skills` and `npm` packages, a compromised `skill` or `npm` dependency could unknowingly infect your system through malicious package updates. *(like the Axios compromise of early 2026)*


### ⚠️ Command Injection Vulnerabilities 
In OpenClaw, command injection isn't just a coding bug; it’s an architectural risk known as the Lethal Trifecta. 
 - **Access to Untrusted Content:** The agent can read emails, browse the web, or monitor messaging apps.
 - **Autonomous Tool Use:** The agent has a "Skill" to run shell commands (bash, zsh, terminal).
 - **Persistence:** The agent maintains a long-term memory and can run commands and crons (scheduled tasks) in the background.

**🔑 SECRETS WARNING:** OpenClaw stores API tokens and session keys in plaintext within `~/.openclaw/credentials/`. Never include these in your Git commits or share your `SOUL.md` file if it contains sensitive environmental details.

**🕸️ Potential Spread:** Without proper network segmentation, a compromised OpenClaw instance may be leveraged as a pivot point for unauthorized access to other internal network devices and network-attached storage.

##
## 🧪 OpenClaw Setup Recommendations for Securing a Locally Hosted AI Lab Environment 🧪

## 1. 📡 Network Segmentation & Isolation - 💣 *The Blast Radius Control*  💣
Before proceeding, it is important to isolate the OpenClaw system from the rest of the LAN (local area network). This helps mitigate the threat of internal pivoting 
and contains potential adversarial activity within the isolated network segment.

### ( Option A ) - Network Segmentation the Proper Way - VLANs
### 🛡️ Put the OpenClaw system on its own dedicated and isolated VLAN. 
By enforcing strict network segmentation, this configuration mitigates the threat of internal pivoting and contains potential adversarial activity within the isolated 
network segment. 
- **VLAN setup and creation is outside the scope of this guide.**
  - The implementation of Virtual LANs (VLANs) typically requires dedicated, prosumer-grade firewall hardware, such as those offered by UniFi or pfSense/Netgate, due to the complex Layer 2 and Layer 3 traffic management involved.
- **Isolation via Docker is not recommended as docker will cause a performance hit to the LLMs. (~10-20% slower)**

##

### ( Option B ) - Network Segmentation the Cheap & Easy Way - NAT in a Y Configuration 
In this configuration, your ISP router acts as the "Base," and your two personal routers (Lab and Personal) are plugged into the ISP router's LAN ports.

**NAT as a Firewall:** Consumer routers use Network Address Translation (NAT). By default, NAT allows outgoing requests but blocks all incoming requests. A device in the Lab network cannot initiate a connection to a device in the Personal network because the Personal Router's firewall will see it as "unsolicited traffic" from the outside and drop it.

### 🚨 The Risks with using NAT in a Y Configuration as Network Segmentation 🚨
- **The "Upstream" Exposure:** Both routers see the ISP router's network as the "Internet" (WAN). If an AI agent in your Lab network is compromised, it can still scan the ISP router’s subnet. If you have a printer or a guest laptop connected directly to the ISP router, the Lab agent can see and attack them because they are "upstream."
- **Double NAT Issues:** Running two routers deep can cause "Double NAT," which breaks certain types of traffic like VOIP, online gaming, and some VPNs
- **The ISP Router's "Admin Page" Risk:** By default, Router B and Router C can both "see" the login page of the ISP Router. If a malicious AI agent gains access to your Lab router/network, it might try to brute-force the ISP router to gain control of the entire network.

![alt text](https://github.com/ArronJablonowski/OpenClawNotes/blob/main/img/Y_Networking_Diagram_small.png)

## 2. 🧑‍💻 Admin User Creation
If the system has been used for personal use, create a brand new admin account and delete any prior personal use accounts from the system entirely. Best practice would 
be to fully re-image/reload MacOS for a fresh system.

- **Make an Admin User:** This account will be used to run admin tasks only, and NOT LLMs.
  - Login to the new account via the GUI and go through the initial setup screens. DO NOT sign into your AppleID. Skip it. 
  - Set a static (fixed) IP address. *ie. Setup a DHCP resevation, etc.*
  - Disable rotating MAC if using a DHCP reservation and WiFi *(Settings > Network > WiFi > SSID's (...) > Network Settings > Private WiFi address > Set to: "Fixed")* You may have to toggle off "Limit IP address Tracking" first, and then click 'OK'. Then toggle off WiFi, and toggle it back on to access "Private Wi-Fi Address", and set it to "Fixed". 
  - Set Screen timeout *(Settings > Lock Screen)*
  - Set Energy settings *(Settings > Energy: -- turn low power mode off -- prevent automatic sleeping when the display is off should be turned on -- Start up 
automatically after a power failure should be turned on)*

## 3. 🛰️ Allow Remote Access
- **Enable SSH** *(Settings > General > Sharing > Remote Login: toggle on)*
  - Grant **Full Disk Access** to Terminal/SSH.
  - Setup SSH key pairs for the admin user.

## 4. 🤖 LLM / AI User Creation
- **Create a secondary User Account** with (*temporary*) admin privileges *(Settings > Users & Groups > Add User)*
  - *Note: Admin rights will be removed after tool installation for security / principle of least privilege.*
  - Restart the system after granting 'Admin' privileges to the new LLM/AI user to ensure all permissions are fully applied.
- Now log into the new **secondary account** via the GUI; skip Apple ID sign-in for security and privacy.
  - *Note: The following installs and commands will happen under this secondary account.*

## 4.1 Install Homebrew - Package Manager 
```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
- Pay attention to the commands that Homebrew asks you to run once the install completes.
- It will look something like the following: 
```zsh
    echo >> ~/.zprofile
    echo 'eval "$(/opt/homebrew/bin/brew shellenv zsh)"' >> ~/.zprofile
    eval "$(/opt/homebrew/bin/brew shellenv zsh)"
```

## 4.2 Install Brew Utilities
```zsh
brew install git python3 node htop tmux p7zip asitop repomix
brew install --cask obsidian
brew install ollama
brew services start ollama
```

## 4.3 Install Stats to Monitor System Temps
```zsh
mkdir ~/Applications && brew install --cask --appdir=~/Applications stats
```
- **Open Stats in the GUI.** (command + spacebar > *type:* 'Stats')
- Durring initial setup - tell Stats to "Start the application automatically when starting your Mac" 
- Durring initial setup - Select "Do everything siliently in the background." 
- Click on the (new) stats that appeared in Mac's top bar, and then click the Gear icon to access Stats Settings. 
    - Click 'CPU' > Ensure the top right toggle is on 
    - Click 'GPU' > Ensure the top right toggle is on
    - Click 'RAM' > Ensure the top right toggle is on
    - Click 'Sensors' > Toggle "Hottest CPU" & "Hottest GPU" on
- Click Gear icon in Settings Window (lower left)
    - Set Tempature to Celsius
    - Toggle on "Show icon in dock"
    - Toggle on **"Start at login"** if not already set. 

## 4.4 Install Mac Fan Control *(if your Mac has a fan)*
```zsh
brew install --cask macs-fan-control
```
- **Open Mac Fan Control in the GUI.** (command + spacebar > *type:* 'Fan')
- Set Exhaust control to Custom > Select "Sensor-based value" > CPU Core Average
    - Set Temperature, that the fan speed will start to increse from to 50c
    - Set Maximum temperature to 75c. This will spin up the fan before the Mac starts getting too hot.
- Click Preferences
    - Under 'General' > select all 3 options *(Autostart minimized..., Check for updates..., Show icon in dock)*
    - Under 'Menu bar display' > Set -- Icon:Monochrome, Fan:Exhaust, Sensor:CPU Core Average 

## 4.5 Pull Gemma4 / or another LLM to act as the "Brain" for OpenClaw 
*Please note the LLM used should be compatible with the hardware you're running it on. If Gemma4 is not compatible, please choose a model that works for your hardware. Ask Gemini, etc. which model is best for your hardware if you're unsure.*
```zsh
ollama pull gemma4
```
- Test ollama/gemma4 
```zsh
ollama run gemma4 "Tell me a joke."
```
 - *You should see it thinking, and then it should give you some ("joke") output.*
 - *If you see output, then ollama/gemma4 is working.*

## 4.6 Modify the Terminal's Prompt 
This will be a visual cue to help identify when you're in the LLM/AI user's terminal, and help prevent entering commands into the wrong terminal window. 
```zsh
nano ~/.zshrc
```
- Add the following to the end of the .zshrc file:
```zsh
# This function prepends the 'Brain' emoji (🧠) to the prompt for a visual cue/awareness.
preprompt_brain() {
    # Emoji goes here: 
    echo -e "🧠 " 
}
# Hook the function into the Zsh prompt drawing routine.
precmd() {
    preprompt_brain
}
```
- Reload the zshrc file
```zsh
source ~/.zshrc
```

## 4.7 Harden ssh to only allow keypair auth 
Ensure you've already setup keypairs for both users. 
```zsh
sudo nano /etc/ssh/sshd_config.d/00-disable-passwords.conf
```
- Add the following to the 00-disable-passwords.conf file: 
```zsh
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
PermitEmptyPasswords no
```

## 4.8 Modify tmux config for ez mode (like scrolling with your mouse or trackpad) 
```zsh
nano ~/.tmux.conf
```
- Add the following to the .tmux file:
```zsh
# Enable mouse support (clicking and scrolling)
set -g mouse on

# Start window and pane numbering at 1 (instead of 0)
set -g base-index 1
setw -g pane-base-index 1

# Don't rename windows automatically (keep your labels)
set-option -g allow-rename off
```

## 4.9 sudo visudo  
- vi editor commands cheat sheet: 
  - press 'i'              *enter insert mode*
  - press '{esc} : q!'     *quit vi, without writing changes to the file*
  - press '{esc} : wq'     *write the changes, and then quit vi*
  - press '{esc} : w'      *Save (write) the file but stay in the editor*
  - press 'ZZ'             *while in Command Mode, no colon needed - Save and quit q*
    
Add commands to the sudoer's file so your secondary LLM/AI user will be allowed to run some commands as root.
```zsh
sudo visudo
```
- Add the following to the end of the file, and replace *'lobster_user'* with your LLM/AI user's name: 
```zsh
lobster_user ALL=(ALL) NOPASSWD: /usr/bin/powermetrics
lobster_user ALL=(ALL) NOPASSWD: /opt/homebrew/bin/asitop
lobster_user ALL=(ALL) NOPASSWD: /Users/lobster_user/.local/bin/uvx fluidtop
```

## 4.10 Install OpenClaw 🦞 
```zsh
curl -fsSL https://openclaw.ai/install.sh | bash
```
As you go through the installer script, keep gemma4 as default and enable 'command-logger' & 'session-memory'.

## 5. 🧑‍💻 Log out of your LLM/AI User and log into your Admin User again
- **Remove Admin/Sudo from the LLM/AI User:** Now that everything is installed and running properly under the LLM/AI user, we will revoke the user's admin/sudo access. 

  - Remove Admin (settings > Users & Groups > *click 'i' next to LLM/AI user's name* and toggle off the "allow this user to administer this computer")
  - Reboot the system so these changes take effect.
  - Verify the LLM/AI user no longer has Admin privileges. 

## 6. 🤖 Login to LLM/AI User Account & Tell OpenClaw to Run a Security Audit
```zsh
openclaw security audit --deep
```

If you want to fix any issues that are found by the audit:
```zsh
openclaw security audit --fix 
```


### 7. 🔒 Hardening the OpenClaw Agent (Zero-Trust Deployment)

You must treat an autonomous agent as **untrusted code execution**. By default, OpenClaw has the same permissions as your `LLM/AI` user. If the agent is "prompt 
injected" while reading a malicious file, email, or webpage, it could trick the system into running malicious commands and/or compromise the system. 

### **Implement "Least Privilege" Command Filtering**
OpenClaw’s `exec` tool is a powerful vector. Restrict it to a "Default Deny" posture by configuring an allow list of approved binaries.

1. Edit your tool policy: `nano ~/.openclaw/openclaw.json`
2. Update the `tools.exec` section:
```json
{
  "tools": {
    "exec": {
      "host": "gateway",
      "security": "allowlist",   // Options: "allowlist", "deny", or "full"
      "ask": "on-miss",          // Options: "always", "on-miss", or "off"
      "allowlist": [
        "/opt/homebrew/bin/rg",
        "/usr/bin/git",
        "/usr/local/bin/birtha*", // Wildcards are supported for your framework
        "/usr/bin/curl"
      ],
      "deny": [
        "sudo",
        "rm",
        "chmod"
      ]
    }
  }
}
```
Modify the allow and deny list as needed. Any binary not listed should prompt you before executing.

## 8. 🧦 SOC-Grade Security Monitoring 
To achieve a resilient SOC-Grade security posture, it is essential to implement comprehensive telemetry monitoring across both host and network layers. This dual-visibility framework provides the necessary context to identify early Indicators of Compromise (IoCs) and facilitates rapid incident containment.

   - Endpoint Observability: Deploy enterprise-grade EDR and log-forwarding solutions—such as Wazuh *(free)* or Elastic Agent *(also free)* to capture granular process-level telemetry and system events.

   - Network Traffic Analysis (NTA): Utilize port mirroring (SPAN/TAP) to ingest raw traffic into a high-fidelity sensor. Implementing a Security Onion or Zeek instance allows for deep packet inspection and automated protocol analysis." 

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
- openclaw models status 
- openclaw models set ollama/gemma4
- openclaw gateway restart 
- openclaw --help

OpenClaw stores its files in a hidden folder ```~/.openclaw```

Common ollama commands: 
- ollama list
- ollama ps
- ollama pull <model_name>
- ollama run <model_name>
- ollama rm <model_name>

repomix: 
- cd into repo's root and run 'repomix' - this will output a file to that can be fed to AI models

```zsh
# Example feeding repomix-output to an AI model for analysis
cat repomix-output.xml | ollama run qwen2.5-coder:7b "I have attached my repository in XML format. Please analyze the logic flow between the main entry script and the 
sub-modules."
```

OpenClaw Docs: 
- [OpenClaw Gateway Security](https://docs.openclaw.ai/gateway/security)
- [OpenClaw Gateway Remote](https://docs.openclaw.ai/gateway/remote)
- [OpenClaw Web Control UI](https://docs.openclaw.ai/web/control-ui)

OpenClaw FAQ: 
- [OpenClaw Start FAQ](https://docs.openclaw.ai/start/faq)

OpenClaw: 
- [OpenClaw Documentation](https://docs.openclaw.ai/)

ollama: 
- [ollama.com](https://ollama.com/)

OhMyZsh: 
- [ohmyz.sh](https://ohmyz.sh/)

##

# Optinal Extras 

## Install Oh My Zsh - Shell Extension 
Oh My Zsh simplifies the process of customizing your terminal by providing over 300 optional plugins for tools like Git and Docker, along with 150+ visual themes. Essentially, it transforms a plain command-line interface into a more powerful, user-friendly environment with features like advanced tab completion and syntax highlighting.
```sh
# Install Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## Install 'uv' and fluidtop to Monitor Performance
```zsh 
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env sh, bash, zsh
uv tool install fluidtop
sudo uvx fluidtop
```

##

## AppleID Cleanup - *If you MUST use an existing user's account...*
If you must use a previously used personal account for some reason, here are some basic cleanup steps. 

- **⚠️ Log Out of Your AppleID and Cleanup Synced Data:** This will limit the impact if your system gets compromised, and the attack is able to elevate privileges. 
  - If you've ever logged into your AppleID using this account or any account on the system, log out. *(Settings > click on your AppleID > click Log Out)*
  - When promoted uncheck all the boxes to save cloud data locally. Delete all of your pictures, and personal data from your user's directories.
  - Check you Keychain & Passwords app, and delete out any stored credentials, etc. that are not needed for your lab environment. *(!! Ensure you're logged out of your AppleID 
first so you don't delete/modify your synced Keychain)* 
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

*Note:* Don't forget to clean up other areas of your user profile - local files, browser files (browser history, cookies, etc.), and any other sensitive information you would 
want to keep private.

# How to Uninstall OpenClaw 
```zsh
openclaw uninstall --all --yes
launchctl bootout gui/$UID/ai.openclaw.gateway
npm rm -g openclaw
pnpm remove -g openclaw
rm -rf ~/.openclaw
rm -f ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```
