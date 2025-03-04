---
title: Install Docker Windows Engine without Docker Desktop
date: 2025-03-04
categories: [Docker, Windows]
tags: [docker, windows, installation]
comments: true
---

# Install Docker Windows Engine without Docker Desktop

This guide will help you install Docker Windows Engine without using Docker Desktop, which will avoid the need for the Docker Desktop License. Follow these steps:

1. **Download the Latest Version**
   - Visit [Docker Windows Engine](https://download.docker.com/win/static/stable/x86_64/) to download the latest version.

2. **Extract the Zip File**
   - Extract the downloaded zip file to `C:/Docker`.

3. **Update System Environment Path**
   - Add the `C:/Docker` directory to the System Environment Path to allow easy access to Docker commands from any terminal.

4. **Register Docker as a Service**
   - Navigate to `C:/Docker` and run the following command in admin mode to register Docker as a service:
     ```sh
     dockerd --register-service
     ```

5. **Start the Docker Service**
   - Use the following commands to start the Docker service:
     ```sh
     Get-Service docker
     Start-Service docker
     ```

6. **Verify Installation**
   - Run the following commands to ensure Docker is installed correctly:
     ```sh
     docker --version
     docker info
     docker run hello-world
     ```

7. **Troubleshoot Network Issues**
   - If you encounter network issues, follow these steps:
     - Set the environment variable `DOCKER_HOST` to `tcp://127.0.0.1:2375`.
     - Update or add the configuration in `C:\ProgramData\docker\config\daemon.json` by running the following command:
       ```sh
       notepad C:\ProgramData\docker\config\daemon.json
       ```

     - Add the following content to the `daemon.json` file:
       ```json
       {
         "hosts": ["tcp://0.0.0.0:2375", "npipe://"]
       }
       ```
     - Restart the service:
       ```sh
       Stop-Service docker
       Start-Service docker
       ```
