# Plan: Sabia - Your Personal Local LLM Server

This document outlines the plan to set up **Sabia**, your secure personal Large Language Model (LLM) server, on your desktop PC. This will make it accessible by other devices like your MacBook Air and iPhone.

## 1. The Overall Plan

This plan is divided into phases for a structured approach to building **Sabia**:

### Phase 1: Core LLM Server Setup (Sabia - Desktop PC)

1. **Install Ollama:**
    
    - Download and install Ollama on your Windows desktop PC.
        
    - Pull an initial model (e.g., `ollama pull llama3:8b-instruct`). You can later add others that Sabia will use.
        
    - Refer to the [Ollama Library](https://ollama.com/library "null") for more models.
        
2. **Choose and Install a Web UI for Sabia:**
    
    - **Recommended:** Open WebUI (formerly Ollama WebUI). This will be the primary interface for interacting with Sabia.
        
    - **Installation (Open WebUI with Docker - Recommended):**
        
        - Install Docker Desktop for Windows.
            
        - Run the Open WebUI container:
            
            ```
            docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui-for-sabia --restart always ghcr.io/open-webui/open-webui:main
            ```
            
            _(Adjust port `3000` if necessary. The container is named `open-webui-for-sabia` for clarity)_
            
    - **Alternatives:** LibreChat, custom-built UI.
        
3. **Initial Local Test of Sabia:**
    
    - Open `http://localhost:3000` (or your chosen port) in a browser on your desktop PC.
        
    - Verify you can interact with the LLM via the Web UI – your first conversation with Sabia!
        

### Phase 2: Local Network Access (MacBook Air to Sabia)

1. **Find Sabia Server's (Desktop PC) Local IP Address:**
    
    - Use `ipconfig` in Command Prompt on the Windows PC. Note the IPv4 address (e.g., `192.168.1.X`).
        
2. **Configure Windows Firewall for Sabia Access:**
    
    - Open "Windows Defender Firewall with Advanced Security."
        
    - Create a new "Inbound Rule":
        
        - Type: Port
            
        - Protocol: TCP
            
        - Specific local ports: (e.g., `3000` - the port for Open WebUI)
            
        - Action: Allow the connection
            
        - Profile: Private (and/or Domain if applicable)
            
        - Name: (e.g., "**Sabia Access**" or "**OpenWebUI for Sabia**")
            
3. **Test from MacBook Air:**
    
    - Ensure MacBook is on the same Wi-Fi network as the Sabia server.
        
    - Navigate to `http://<sabia_server_local_ip>:<port>` in a browser on the MacBook.
        

### Phase 3: Security Considerations for Sabia

- **Keep Sabia Local:** Avoid port forwarding on your internet router unless absolutely necessary and understood.
    
- **Strong Wi-Fi Password:** Secure your home Wi-Fi, which protects Sabia's local network.
    
- **Web UI Authentication:** Utilize Open WebUI's user management features with strong passwords for accessing Sabia.
    
- **Ollama Host Configuration (Advanced):**
    
    - Ollama 0.1.32+ defaults to listening on `127.0.0.1`.
        
    - If Open WebUI (in Docker) cannot reach Ollama, you might need to set `OLLAMA_HOST=0.0.0.0` (or your PC's LAN IP) as an environment variable for Ollama. This allows Ollama to accept connections from other addresses on your local network, including the Docker container for Sabia's web interface.
        
    - The `--add-host=host.docker.internal:host-gateway` in the Docker command helps the container find services running on the host.
        
- **Regular Updates:** Keep Ollama, Open WebUI, Docker, and your OS updated to protect Sabia.
    

## 2. Suggested Repository Structure for Sabia

A Git repository can help manage configurations and notes for your **Sabia** project:

```
Sabia-Server/
├── .gitignore
├── README.md              # Overview of Sabia, setup, quick start
│
├── ollama_setup/
│   ├── notes.md           # Ollama install notes, models for Sabia, env vars
│
├── web_ui_config/
│   ├── docker-compose.yml # Optional: for Docker setup for Sabia's WebUI
│   └── open-webui/
│       └── config_notes.md  # Web UI settings for Sabia
│
├── scripts/               # Optional: utility scripts for Sabia
│   ├── start_sabia.sh
│   └── backup_sabia_data.sh
│
└── docs/
    ├── networking_setup.md # IP, firewall details for Sabia
    ├── security_policy.md  # Security guidelines for Sabia
    └── iphone_access.md    # iPhone access setup for Sabia
```

**Example `.gitignore`:**

```
*.log
/web_ui_config/open-webui/data/ # If mapping data volume here
__pycache__/
*.env
```

## 3. Including Chat History in Sabia

- **Yes, this is a standard feature of Open WebUI, which serves as Sabia's interface.**
    
- Open WebUI stores conversations, typically in the Docker volume (`open-webui:/app/backend/data`).
    
- The Web UI manages sending conversation context to the Ollama API, enabling "memory" for Sabia during conversations.
    

## 4. Plan for Chatting with Sabia from Your iPhone

This enables access to **Sabia** when your iPhone is not on your home Wi-Fi.

**Recommended Method: Tailscale (Mesh VPN)**

1. **Install Tailscale:**
    
    - On your **Desktop PC (Sabia Server)**.
        
    - On your **iPhone (Client)**.
        
    - Log in with the same account on both.
        
2. **How it Works:**
    
    - Creates a secure, private network (tailnet) between your devices.
        
    - Your Sabia server gets a stable Tailscale IP (e.g., `100.X.X.X`).
        
3. **Access Sabia from iPhone:**
    
    - Open a browser on your iPhone.
        
    - Navigate to `http://<sabia_server_tailscale_ip>:<port_for_web_ui>`.
        

**Alternative Methods:**

- **Traditional VPN:** Set up a VPN server on your home router or a dedicated device. Connect your iPhone to this VPN. Access Sabia using the desktop's _local_ IP.
    
- **Cloudflare Tunnel:**
    
    - Install `cloudflared` on your Sabia server.
        
    - Create a tunnel to your local Web UI (e.g., `http://localhost:3000`).
        
    - Access Sabia via a `something.trycloudflare.com` URL.
        
    - **Crucial:** Use Cloudflare Access to add an authentication layer to the tunnel.
        

**Important for iPhone Access to Sabia:**

- Ensure the Web UI (Open WebUI) is mobile-responsive.
    
- Be mindful of battery and data usage on cellular.
    

This structured plan should guide you through setting up **Sabia**. Enjoy building and interacting with your wise AI companion!