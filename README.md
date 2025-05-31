# Sabia-Server

**Sabia** is a project to create a personal, locally-hosted Large Language Model (LLM) server. This server will be accessible from other devices on the local network, such as a MacBook Air and iPhone, and prioritizes data privacy and control.

The core idea is to leverage open-source LLMs (via Ollama) and a user-friendly web interface (like Open WebUI) to provide a secure and customizable AI experience.

## Project Goals

*   Set up a personal LLM server (Ollama + Open WebUI) on a desktop PC (Windows, potentially using WSL2).
*   Enable SSH access to the server PC from other devices on the local Wi-Fi.
*   Enable access to the Open WebUI (via URL) and Ollama API (for programmatic requests) from other devices on the local network.
*   Explore secure remote access options (e.g., Tailscale) for scenarios beyond the local Wi-Fi.
*   Maintain data sovereignty by keeping all processing and data local.
*   Investigate potential commercial applications for similar private AI solutions for businesses.

## Quick Start & Setup

This repository contains configurations and documentation to guide the setup process.

1.  **Core LLM Server Setup (Desktop PC - Windows, consider WSL2 for a Linux environment):**
    *   Install Ollama (see `ollama_setup/notes.md`).
        *   Ensure Ollama is configured to be accessible from other devices on the network (e.g., by setting `OLLAMA_HOST=0.0.0.0` if running directly or ensuring Docker networking/WSL port forwarding allows access).
    *   Install a Web UI, preferably Open WebUI via Docker (see `web_ui_config/docker-compose.yml` and `web_ui_config/open-webui/config_notes.md`).
        *   Ensure the Web UI is accessible from other devices on the network (similar network configuration as Ollama).
2.  **Local Network Access & SSH:**
    *   **SSH Setup:**
        *   Install and configure an SSH server on the desktop PC (e.g., OpenSSH Server on Windows or within WSL).
    *   **Firewall & Port Forwarding:**
        *   Configure firewall rules on the server PC to allow incoming connections for SSH (typically port 22), Ollama (typically port 11434), and the Open WebUI (e.g., port 3000 or your chosen port).
        *   If using WSL2 for Ollama/WebUI, set up port forwarding on the Windows host to forward traffic from the Windows IP to the WSL2 instance for the respective services.
    *   Refer to `docs/networking_setup.md` for more detailed guidance.
3.  **Security:**
    *   Review security considerations, especially for SSH and network-exposed services (see `docs/security_policy.md`).
4.  **(Optional) iPhone Access (Beyond Local Wi-Fi):**
    *   Set up remote access if needed (see `docs/iphone_access.md`).

## Repository Structure

*   `.gitignore`: Specifies intentionally untracked files that Git should ignore.
*   `README.md`: This file - overview of Sabia, setup, quick start.
*   `ollama_setup/`: Contains notes and configurations related to Ollama.
    *   `notes.md`: Ollama installation notes, models for Sabia, environment variables.
*   `web_ui_config/`: Contains configurations for the Web UI.
    *   `docker-compose.yml`: Optional Docker Compose file for setting up Open WebUI.
    *   `open-webui/config_notes.md`: Notes on Web UI settings for Sabia.
*   `scripts/`: Optional utility scripts for managing Sabia.
    *   `start_sabia.sh`: Script to start Sabia services.
    *   `backup_sabia_data.sh`: Script to backup Sabia data.
*   `docs/`: Contains detailed documentation.
    *   `networking_setup.md`: IP, firewall details for Sabia.
    *   `security_policy.md`: Security guidelines for Sabia.
    *   `iphone_access.md`: iPhone access setup for Sabia.
*   `Sabia - Your Personal Local LLM Server.md`: The main planning document for the personal Sabia server.
*   `Commercial Potential - Sabia-like Local AI Solutions for Businesses.md`: Explores commercial avenues for similar local AI solutions.

Refer to `Sabia - Your Personal Local LLM Server.md` for the detailed plan and `Commercial Potential - Sabia-like Local AI Solutions for Businesses.md` for business-related explorations.
