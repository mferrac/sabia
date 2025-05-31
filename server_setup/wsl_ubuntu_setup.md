# Server Setup on Windows with WSL2 (Ubuntu)

This guide outlines the steps to set up your Sabia LLM server environment on a Windows machine using Windows Subsystem for Linux 2 (WSL2) with an Ubuntu distribution.

## 1. Prerequisites

*   Windows 10 (version 2004 or later) or Windows 11.
*   WSL2 enabled on your Windows machine.
    *   You can enable it by running `wsl --install` in an administrator PowerShell or Command Prompt. This will also install Ubuntu by default. If you have an older version, search for enabling "Virtual Machine Platform" and "Windows Subsystem for Linux" features.
*   Ubuntu distribution installed from the Microsoft Store (or installed via `wsl --install`).
*   Docker Desktop for Windows installed, with WSL2 integration enabled:
    *   Download and install from [Docker's website](https://www.docker.com/products/docker-desktop/).
    *   In Docker Desktop settings > Resources > WSL Integration, ensure your Ubuntu distribution is enabled.

## 2. Initial WSL/Ubuntu Setup

1.  **Open your Ubuntu terminal.**
2.  **Update and upgrade packages:**
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

## 3. Networking Configuration (Crucial for LAN Access)

Services running inside WSL2 have their own IP addresses. To make them accessible from your Mac or other devices on your local network, you need to:

*   **Find your Windows host IP address:**
    *   Open Command Prompt (`cmd`) on Windows and type `ipconfig`. Note the IPv4 address of your active network adapter (e.g., `192.168.1.100`). This is `<YourWindowsIP>`.
*   **Find your WSL2 instance's IP address:**
    *   In your Ubuntu (WSL2) terminal, type:
        ```bash
        ip addr show eth0 | grep "inet\s" | awk '{print $2}' | cut -d/ -f1
        ```
    *   Note this IP address (e.g., `172.X.X.X`). This is `<YourWSLIP>`.

*   **Port Forwarding on Windows Host:**
    You need to forward ports from your Windows host IP to your WSL2 instance IP for Ollama and Open WebUI. Open **PowerShell as Administrator** on Windows and run the following commands.

    *   **For Ollama (default port 11434):**
        ```powershell
        netsh interface portproxy add v4tov4 listenport=11434 listenaddress=<YourWindowsIP> connectport=11434 connectaddress=<YourWSLIP>
        ```
        Replace `<YourWindowsIP>` and `<YourWSLIP>` with the actual IPs.
    *   **For Open WebUI (default port 3000, as per `docker-compose.yml` mapping to 8080 internally):**
        ```powershell
        netsh interface portproxy add v4tov4 listenport=3000 listenaddress=<YourWindowsIP> connectport=3000 connectaddress=<YourWSLIP>
        ```
        *Note: If Open WebUI inside WSL2 (via Docker) is set to listen on port 8080, and your `docker-compose.yml` maps `3000:8080`, you'd forward to the port Docker exposes on the WSL2 IP, which would be 3000 if you run the docker-compose directly in WSL. If Open WebUI itself inside the container listens on 8080, and Docker maps that to WSL's 3000, then `connectport` should be 3000.*
        *For simplicity with the provided `docker-compose.yml`, assuming it runs in WSL and exposes port 3000 on the WSL IP, the command above is correct.*

    *   **To view existing port forwards:**
        ```powershell
        netsh interface portproxy show all
        ```
    *   **To delete a port forward (if needed):**
        ```powershell
        netsh interface portproxy delete v4tov4 listenport=11434 listenaddress=<YourWindowsIP>
        ```

*   **Windows Firewall Configuration:**
    Create inbound rules in Windows Defender Firewall to allow traffic on the ports you are forwarding (e.g., 11434 for Ollama, 3000 for Open WebUI) for your **Private** network profile.
    1.  Open "Windows Defender Firewall with Advanced Security".
    2.  Go to "Inbound Rules" -> "New Rule...".
    3.  Type: Port, Protocol: TCP.
    4.  Specific local ports: `11434, 3000` (or add separate rules).
    5.  Action: Allow the connection.
    6.  Profile: Check "Private" (ensure your Wi-Fi is set to Private network profile).
    7.  Name: e.g., "Sabia LLM Server Access".

## 4. SSH Server Setup (Optional, for terminal access to WSL)

1.  **Install OpenSSH Server in Ubuntu (WSL):**
    ```bash
    sudo apt install openssh-server
    ```
2.  **Configure SSHD:**
    *   Edit `/etc/ssh/sshd_config` (e.g., `sudo nano /etc/ssh/sshd_config`):
        *   Set `PasswordAuthentication yes` (if you want to use passwords, ensure they are strong).
        *   Ensure `Port 22` is uncommented (or choose a different port).
        *   Optional: `ListenAddress 0.0.0.0`
    *   Restart SSH service:
        ```bash
        sudo service ssh restart
        ```
    *   Or, if using systemd (may require enabling it in WSL): `sudo systemctl restart ssh`
3.  **Port Forwarding for SSH on Windows Host:**
    *   In an **Administrator PowerShell**:
        ```powershell
        netsh interface portproxy add v4tov4 listenport=2222 listenaddress=<YourWindowsIP> connectport=22 connectaddress=<YourWSLIP>
        ```
        (Using port `2222` on Windows to avoid conflict if Windows host already has an SSH server on port 22).
4.  **Firewall Rule for SSH on Windows Host:**
    *   Add an inbound rule for TCP port `2222` (or your chosen `listenport`) for your Private network.

## 5. Install Ollama and Open WebUI

Follow the instructions in `server_setup/common_ollama_webui_setup.md`. Remember that Docker commands (like `docker-compose up -d`) should be run from within your Ubuntu (WSL2) terminal.

## 6. Testing Access from another PC (e.g., your Mac)

*   **Open WebUI:** `http://<YourWindowsIP>:3000`
*   **Ollama API:** `http://<YourWindowsIP>:11434/api/tags` (Example endpoint)
*   **SSH (if configured):** `ssh your_ubuntu_user@<YourWindowsIP> -p 2222`

This setup ensures that your Sabia server runs within the Linux environment provided by WSL2, and with the correct port forwarding and firewall rules, it becomes accessible to other devices on your local network.
