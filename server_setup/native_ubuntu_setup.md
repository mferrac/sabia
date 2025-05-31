# Server Setup on Native Ubuntu

This guide outlines the steps to set up your Sabia LLM server environment directly on a machine running Ubuntu (e.g., a dedicated server or a PC with Ubuntu installed).

## 1. Prerequisites

*   A machine with Ubuntu Server or Desktop installed and updated.
*   Sudo privileges.
*   Basic familiarity with the Linux command line.

## 2. Initial Server Setup

1.  **Update and upgrade packages:**
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```
2.  **Install Docker:**
    *   Follow the official Docker installation guide for Ubuntu: [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
    *   Add your user to the `docker` group to run Docker commands without `sudo` (requires logout/login or new shell):
        ```bash
        sudo usermod -aG docker ${USER}
        newgrp docker
        ```
3.  **Install Docker Compose (Plugin):**
    *   Follow the official guide: [Install Docker Compose](https://docs.docker.com/compose/install/) (Ensure you install the Compose plugin, typically `docker-compose-plugin`).

## 3. Networking Configuration

*   **Find your Ubuntu server's IP address:**
    ```bash
    ip addr show | grep "inet\s" | grep -v "127.0.0.1" | awk '{print $2}' | cut -d/ -f1
    ```
    Note this IP address (e.g., `192.168.1.101`). This is `<YourUbuntuServerIP>`.

*   **Firewall Configuration (using `ufw`):**
    `ufw` (Uncomplicated Firewall) is a common firewall for Ubuntu.
    1.  **Install `ufw` (if not already installed):**
        ```bash
        sudo apt install ufw
        ```
    2.  **Allow SSH (crucial if you're accessing remotely):**
        ```bash
        sudo ufw allow ssh  # Or sudo ufw allow 22/tcp
        ```
    3.  **Allow Ollama port (default 11434):**
        ```bash
        sudo ufw allow 11434/tcp
        ```
    4.  **Allow Open WebUI port (default 3000, as per `docker-compose.yml`):**
        ```bash
        sudo ufw allow 3000/tcp
        ```
    5.  **Enable `ufw`:**
        ```bash
        sudo ufw enable
        ```
        Confirm with `y`.
    6.  **Check status:**
        ```bash
        sudo ufw status verbose
        ```

## 4. SSH Server Setup (Usually enabled by default on Ubuntu Server)

1.  **Install OpenSSH Server (if not present, common on Desktop versions):**
    ```bash
    sudo apt install openssh-server
    ```
2.  **Ensure SSH service is running and enabled:**
    ```bash
    sudo systemctl status ssh
    sudo systemctl enable ssh --now
    ```
3.  **Configuration (Optional):**
    *   Edit `/etc/ssh/sshd_config` (e.g., `sudo nano /etc/ssh/sshd_config`) for custom settings (e.g., changing port, disabling password authentication in favor of keys).
    *   Restart SSH if changes are made: `sudo systemctl restart ssh`

## 5. Install Ollama and Open WebUI

Follow the instructions in `server_setup/common_ollama_webui_setup.md`. Docker commands (like `docker-compose up -d`) should be run from your Ubuntu terminal.

## 6. Testing Access from another PC (e.g., your Mac)

*   **Open WebUI:** `http://<YourUbuntuServerIP>:3000`
*   **Ollama API:** `http://<YourUbuntuServerIP>:11434/api/tags` (Example endpoint)
*   **SSH (if configured):** `ssh your_ubuntu_user@<YourUbuntuServerIP>`

This setup ensures that your Sabia server runs directly on your Ubuntu machine, and with the correct firewall rules, it becomes accessible to other devices on your local network.
