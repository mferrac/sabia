# Server Setup on Native Windows

This guide outlines the steps to set up your Sabia LLM server environment directly on a machine running Windows (e.g., a dedicated server or a PC with Windows installed).

## 1. Prerequisites

*   A machine with Windows 10/11 installed and updated.
*   Administrator privileges.
*   Basic familiarity with Windows Command Prompt or PowerShell.

## 2. Initial Server Setup

1.  **Update Windows:**
    ```powershell
    # Check for and install Windows Updates
    # Go to Settings > Update & Security > Windows Update
    # Or use PowerShell (as Administrator):
    Get-WindowsUpdate -Install -AcceptAll -AutoReboot
    ```

2.  **Install Ollama (Native Windows):**
    *   Download Ollama for Windows from: [https://ollama.com/download](https://ollama.com/download)
    *   Run the installer and follow the installation wizard.
    *   Ollama will automatically install as a Windows service and start running.

3.  **Install Docker Desktop (Optional - for Open WebUI):**
    *   Download Docker Desktop for Windows from: [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
    *   Install and restart your computer when prompted.
    *   Ensure Docker Desktop is running (check system tray).

## 3. Configure Ollama for Network Access

By default, Ollama only listens on localhost (`127.0.0.1`). To make it accessible from other devices on your network:

1.  **Set Environment Variable (Method 1 - Persistent):**
    *   Open "Environment Variables" in Windows:
        - Press `Win + R`, type `sysdm.cpl`, press Enter
        - Click "Environment Variables..." button
        - Under "System variables", click "New..."
    *   Add a new system environment variable:
        - Variable name: `OLLAMA_HOST`
        - Variable value: `0.0.0.0:11434`
    *   Click "OK" and restart your computer.

2.  **Set Environment Variable (Method 2 - Command Line):**
    Open Command Prompt or PowerShell as Administrator:
    ```powershell
    # PowerShell
    [Environment]::SetEnvironmentVariable("OLLAMA_HOST", "0.0.0.0:11434", "Machine")
    
    # Or using setx command
    setx OLLAMA_HOST "0.0.0.0:11434" /M
    ```
    Restart your computer after setting the environment variable.

3.  **Verify Ollama Service:**
    ```powershell
    # Check if Ollama service is running
    Get-Service -Name "Ollama"
    
    # If needed, restart the service
    Restart-Service -Name "Ollama"
    ```

## 4. Networking Configuration

*   **Find your Windows PC's IP address:**
    ```powershell
    # PowerShell method
    Get-NetIPAddress -AddressFamily IPv4 | Where-Object {$_.InterfaceAlias -like "*Wi-Fi*" -or $_.InterfaceAlias -like "*Ethernet*"} | Select-Object IPAddress, InterfaceAlias
    
    # Or simple command
    ipconfig
    ```
    Note the IPv4 address of your active network adapter (e.g., `192.168.1.100`). This is `<YourWindowsServerIP>`.

*   **Windows Firewall Configuration:**
    You need to allow inbound connections on the Ollama and Open WebUI ports.

    **Method 1: Using Windows Defender Firewall GUI:**
    1.  Open "Windows Defender Firewall with Advanced Security" (search in Start menu)
    2.  Click "Inbound Rules" in the left panel
    3.  Click "New Rule..." in the right panel
    4.  Select "Port" → Next
    5.  Select "TCP" → Specific local ports: `11434` → Next
    6.  Select "Allow the connection" → Next
    7.  Check "Private" (your home network) → Next
    8.  Name: "Ollama Server Access" → Finish
    9.  Repeat for port `3000` (Open WebUI) if using Docker setup

    **Method 2: Using PowerShell (as Administrator):**
    ```powershell
    # Allow Ollama port (11434)
    New-NetFirewallRule -DisplayName "Ollama Server Access" -Direction Inbound -Protocol TCP -LocalPort 11434 -Action Allow -Profile Private
    
    # Allow Open WebUI port (3000) - if using Docker setup
    New-NetFirewallRule -DisplayName "Open WebUI Access" -Direction Inbound -Protocol TCP -LocalPort 3000 -Action Allow -Profile Private
    ```

    **Method 3: Using netsh command:**
    ```cmd
    netsh advfirewall firewall add rule name="Ollama Server Access" dir=in action=allow protocol=TCP localport=11434 profile=private
    netsh advfirewall firewall add rule name="Open WebUI Access" dir=in action=allow protocol=TCP localport=3000 profile=private
    ```

## 5. Test Ollama Installation

1.  **Test locally:**
    ```powershell
    # Pull a model
    ollama pull llama3:8b-instruct
    
    # List available models
    ollama list
    
    # Test API endpoint
    curl http://localhost:11434/api/tags
    ```

2.  **Test from another device on the same network:**
    ```bash
    # From another device, replace <YourWindowsServerIP> with your PC's IP
    curl http://<YourWindowsServerIP>:11434/api/tags
    ```

## 6. Install Open WebUI (Optional but Recommended)

If you want a web interface for your Ollama server:

### Option A: Using Docker (Recommended)

1.  **Ensure Docker Desktop is running**

2.  **Navigate to your Sabia project directory:**
    ```powershell
    cd "C:\Users\mferrac\Repositories\sabia\web_ui_config"
    ```

3.  **Run Open WebUI using Docker Compose:**
    ```powershell
    docker-compose up -d
    # Or if using newer Docker Compose plugin:
    docker compose up -d
    ```

4.  **Check container status:**
    ```powershell
    docker ps
    docker logs open-webui-for-sabia
    ```

### Option B: Using Open WebUI Windows Binary (Alternative)

If you prefer not to use Docker, check if Open WebUI has a Windows binary release on their GitHub page.

## 7. Configure Open WebUI to Connect to Ollama

1.  **Access Open WebUI:**
    Open your web browser and go to: `http://localhost:3000`

2.  **Configure Ollama Connection:**
    *   In Open WebUI settings, set the Ollama API URL to:
        - `http://host.docker.internal:11434` (if using Docker)
        - `http://localhost:11434` (if using native installation)

3.  **Create your first account:**
    The first user to register becomes the admin.

## 8. Testing Access from Other Devices

*   **Open WebUI:** `http://<YourWindowsServerIP>:3000`
*   **Ollama API:** `http://<YourWindowsServerIP>:11434/api/tags`
*   **Test with curl from another device:**
    ```bash
    curl http://<YourWindowsServerIP>:11434/api/tags
    ```

## 9. Managing the Server

### Starting/Stopping Ollama Service
```powershell
# Stop Ollama service
Stop-Service -Name "Ollama"

# Start Ollama service
Start-Service -Name "Ollama"

# Restart Ollama service
Restart-Service -Name "Ollama"

# Check service status
Get-Service -Name "Ollama"
```

### Managing Open WebUI (Docker)
```powershell
# Stop Open WebUI
docker-compose down

# Start Open WebUI
docker-compose up -d

# View logs
docker logs open-webui-for-sabia

# Update Open WebUI
docker-compose pull
docker-compose up -d
```

## 10. Troubleshooting

### Common Issues:

1.  **Can't access from other devices:**
    *   Verify firewall rules are set correctly
    *   Check that `OLLAMA_HOST` environment variable is set to `0.0.0.0:11434`
    *   Ensure both devices are on the same WiFi network
    *   Try temporarily disabling Windows Firewall to test

2.  **Ollama service won't start:**
    *   Check Windows Event Viewer for error messages
    *   Restart your computer
    *   Reinstall Ollama if necessary

3.  **Open WebUI can't connect to Ollama:**
    *   Verify Ollama is running: `ollama list`
    *   Check Docker Desktop is running
    *   Verify the Ollama API URL in Open WebUI settings

4.  **Performance issues:**
    *   Ensure your PC has sufficient RAM (8GB+ recommended)
    *   Close unnecessary applications
    *   Consider using smaller models if hardware is limited

## Security Notes

*   This setup only allows access from your local WiFi network
*   Ensure your WiFi network is secured with WPA2/WPA3
*   The firewall rules only allow "Private" network profile (your home network)
*   Never enable port forwarding on your router for these ports unless you understand the security implications
*   Consider setting up user authentication in Open WebUI for additional security

This setup ensures that your Sabia server runs directly on your Windows machine and becomes accessible to other devices on your local network, similar to the Ubuntu setup but adapted for Windows.
