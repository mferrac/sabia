# Common Ollama and Open WebUI Setup

These are the common steps for installing Ollama and Open WebUI, referenced by both WSL/Ubuntu and native Ubuntu setup guides.

## 1. Install Ollama (Native Linux Installation)

1.  **Install Ollama:**
    *   The recommended way to install Ollama on Linux is using their script:
        ```bash
        curl -fsSL https://ollama.com/install.sh | sh
        ```
    *   This will download and run the installation script. It typically sets up Ollama as a systemd service.

2.  **Configure Ollama for Network Access:**
    By default, Ollama might only listen on `127.0.0.1` (localhost). To make it accessible from other devices on your network, you need to configure it to listen on `0.0.0.0` or its specific LAN IP.

    *   **Method 1: Using Environment Variable (Recommended for systemd service)**
        1.  Edit the Ollama systemd service file. The location might vary, but you can often override it:
            ```bash
            sudo systemctl edit ollama.service
            ```
        2.  This will open an override file. Add the following lines:
            ```ini
            [Service]
            Environment="OLLAMA_HOST=0.0.0.0"
            ```
            *Alternatively, you can specify the port too if needed: `Environment="OLLAMA_HOST=0.0.0.0:11434"`*
        3.  Save and exit.
        4.  Reload the systemd daemon and restart Ollama:
            ```bash
            sudo systemctl daemon-reload
            sudo systemctl restart ollama
            ```

    *   **Method 2: Check Ollama documentation for other ways to set `OLLAMA_HOST` if the above doesn't persist or apply correctly for your specific Ollama version or setup.**

3.  **Verify Ollama is Running and Accessible (Locally First):**
    ```bash
    ollama list
    ```
    And from the host machine (or WSL instance itself):
    ```bash
    curl http://localhost:11434/api/tags
    ```
    If you configured `OLLAMA_HOST=0.0.0.0`, you should also be able to test with `curl http://<YourWSLIP_or_UbuntuIP>:11434/api/tags` from within the WSL/Ubuntu environment.

4.  **Pull an Initial Model:**
    ```bash
    ollama pull llama3:8b-instruct  # Or any other model you prefer
    ```
    Example: `ollama pull mistral`

## 2. Install Open WebUI (Using Docker)

This assumes you have Docker and Docker Compose (plugin) installed as per the WSL or native Ubuntu setup guides.

1.  **Navigate to your Sabia project directory where `docker-compose.yml` is located:**
    (Assuming you've cloned or copied your `sabia` repository structure to the server/WSL environment)
    ```bash
    cd /path/to/your/sabia/web_ui_config 
    # Or wherever your docker-compose.yml for open-webui is.
    # The project structure suggests it's in `web_ui_config/docker-compose.yml`
    ```
    If you don't have the `docker-compose.yml` from your project yet, you can use the one provided in `Sabia - Your Personal Local LLM Server.md` or your `web_ui_config/docker-compose.yml`.

    **Example `docker-compose.yml` (from your project plan):**
    ```yaml
    version: '3.8'
    services:
      open-webui:
        image: ghcr.io/open-webui/open-webui:main
        container_name: open-webui-for-sabia
        ports:
          - "3000:8080" # Host port 3000 maps to container port 8080
        volumes:
          - open-webui:/app/backend/data
        add_hosts:
          - "host.docker.internal:host-gateway" # Important for container to reach host services
        restart: always
        # environment:
          # OLLAMA_BASE_URL: http://host.docker.internal:11434 # If Ollama runs on the Docker host (WSL/Ubuntu machine)
          # OLLAMA_BASE_URL: http://<YourWSLIP_or_UbuntuIP>:11434 # More explicit
    
    volumes:
      open-webui: {}
    ```

2.  **Configure Open WebUI to Connect to Ollama:**
    *   The `add_hosts` entry `host.docker.internal:host-gateway` in the `docker-compose.yml` is key. It allows the Open WebUI container to refer to the host machine (your WSL Ubuntu instance or your native Ubuntu server) as `host.docker.internal`.
    *   When you first access Open WebUI, or in its settings, you will configure the Ollama API URL. It should be:
        `http://host.docker.internal:11434`
    *   Alternatively, you can uncomment and set the `OLLAMA_BASE_URL` environment variable in your `docker-compose.yml`:
        ```yaml
        environment:
          OLLAMA_BASE_URL: http://host.docker.internal:11434
        ```
        If `host.docker.internal` doesn't work reliably (it should on modern Docker versions), you might need to use the specific IP address of your WSL2 instance or Ubuntu server that Ollama is listening on (e.g., `http://<YourWSLIP_or_UbuntuIP>:11434`). However, `host.docker.internal` is generally preferred as the IP can change.

3.  **Run Open WebUI using Docker Compose:**
    In the directory containing your `docker-compose.yml`:
    ```bash
    docker-compose up -d
    ```
    Or, if using the compose plugin:
    ```bash
    docker compose up -d
    ```

4.  **Check Container Status:**
    ```bash
    docker ps
    ```
    You should see `open-webui-for-sabia` running.
    ```bash
    docker logs open-webui-for-sabia
    ```
    To see its logs.

## 3. Testing

*   **Locally (within WSL/Ubuntu):**
    *   Open WebUI: `http://localhost:3000`
    *   Ollama API: `curl http://localhost:11434/api/tags`

*   **From another device on your LAN (e.g., your Mac):**
    *   This depends on the port forwarding (for WSL) and firewall rules you've set up.
    *   Open WebUI: `http://<YourWindowsIP_or_UbuntuServerIP>:3000`
    *   Ollama API: `curl http://<YourWindowsIP_or_UbuntuServerIP>:11434/api/tags`

You should now have Ollama and Open WebUI running and accessible.
