# STB HG680P — Docker, CasaOS & Portainer

This repository documents the process of transforming an **HG680P Android TV Box** into a Linux-based **home server** by installing **Docker**, **CasaOS**, and **Portainer** on top of an existing Armbian installation.

The previous stage of the project focused on installing Armbian and enabling the internal RTL8189FS WiFi adapter. This repository continues from that environment and adds the software required to manage and run containerized applications.

The resulting environment provides:

* Docker as the container runtime
* CasaOS as a web-based home server management platform
* Portainer as a Docker management interface
* Docker Compose for container configuration
* A foundation for additional services such as Prometheus, Grafana, Node Exporter, and cAdvisor

---

## Project Overview

The HG680P is originally an Android TV box. After installing Armbian, the device can be repurposed as a low-power Linux server.

This stage focuses on installing three main components:

### Docker

Docker provides the container runtime used to run applications in isolated environments.

### CasaOS

CasaOS provides a simple web-based interface for managing a home server and Docker applications.

### Portainer

Portainer provides a dedicated web interface for managing Docker containers, images, volumes, networks, and other Docker resources.

The combination of these components makes the HG680P easier to use as a small home server without relying entirely on terminal commands.

---

# 1. Prerequisites

This repository assumes that the previous Armbian installation has already been completed.

The HG680P should already have:

* Armbian Server installed
* A configured Linux user
* Network connectivity
* SSH access
* Working Ethernet or WiFi connectivity

The system is accessed remotely through SSH during the installation.

---

# 2. Connect to the STB Through SSH

Before installing Docker, connect to the Armbian system through SSH.

The IP address of the STB can be checked from the Armbian terminal using:

```bash
hostname -I
```

The command returns the IP address assigned to the STB.

From Windows, connect using the built-in OpenSSH client:

```bash
ssh USERNAME@SERVER_IP
```

Replace:

```text
USERNAME
```

with the Linux user configured on the STB.

Replace:

```text
SERVER_IP
```

with the IP address assigned to the STB.

PuTTY can also be used if preferred.

The important requirement is that the STB can be accessed remotely through SSH.

---

# 3. Install Docker

Docker is used as the main container platform for the home server.

The installation uses the official Docker repository rather than relying on an older Docker package provided by the default Ubuntu repository.

---

## 3.1 Update the System Repository

Update the package index:

```bash
sudo apt update
```

Install the required supporting packages:

```bash
sudo apt install ca-certificates curl
```

These packages are required to securely retrieve and configure the Docker repository.

---

## 3.2 Create the Docker Keyring Directory

Create the directory used to store the Docker GPG key:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Download the official Docker GPG key:

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```

Set the required read permissions:

```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

---

## 3.3 Add the Official Docker Repository

Create the Docker repository configuration:

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Update the package index again:

```bash
sudo apt update
```

If no repository error is reported, the Docker repository has been added successfully.

---

# 4. Install Docker Engine

Install Docker Engine and the required Docker components:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

The installation includes:

* Docker Engine
* Docker CLI
* containerd
* Docker Buildx
* Docker Compose Plugin

Docker Compose is important because it will later be used to define and run multi-container applications.

---

# 5. Verify Docker Installation

Check the installed Docker version:

```bash
docker --version
```

The project environment used a Docker 29.x release during the initial implementation.

The exact version may differ depending on when the installation is performed because the official Docker repository is continuously updated.

Check Docker Compose as well:

```bash
docker compose version
```

Both commands should return valid version information.

---

# 6. Test Docker

After installing Docker, run the official `hello-world` image:

```bash
sudo docker run hello-world
```

A successful installation returns a message similar to:

```text
Hello from Docker!

This message shows that your installation appears to be working correctly.
```

This test confirms that Docker can:

1. Contact the Docker daemon.
2. Pull an image from a registry.
3. Create a container.
4. Start the container.
5. Execute the container successfully.

At this point, Docker is ready to run containerized applications.

---

# 7. Install CasaOS

CasaOS is installed after Docker because CasaOS uses Docker as its application runtime.

CasaOS provides a web-based interface for managing the home server and installed applications.

---

## 7.1 Create the CasaOS Directory

Create the installation directory:

```bash
sudo mkdir /opt/casaos
```

Enter the directory:

```bash
cd /opt/casaos
```

The directory provides a dedicated location for the CasaOS installation.

---

## 7.2 Install CasaOS

Run the CasaOS installation script:

```bash
curl -fsSL https://get.casaos.io | sudo bash
```

Wait for the installation process to complete.

The installer will configure the required CasaOS components and display information about the web interface after the installation.

---

# 8. Access the CasaOS Dashboard

After installation, open a browser from a computer connected to the same network as the STB.

Access:

```text
http://SERVER_IP
```

Replace `SERVER_IP` with the IP address assigned to the STB.

If the CasaOS web interface is displayed, the installation has completed successfully.

---

# 9. Initial CasaOS Configuration

During the first access, CasaOS displays an initial configuration page.

Select **Go** and create the administrator account.

The required information includes:

* Username
* Password
* Password confirmation

After completing the form, select **Create**.

The CasaOS dashboard should then be displayed.

CasaOS can now be used as the main web interface for managing applications and services on the home server.

---

# 10. Install Portainer

Portainer is installed to provide a dedicated Docker management interface.

While CasaOS provides a broader home-server management experience, Portainer focuses specifically on Docker administration.

Portainer can be used to manage:

* Containers
* Images
* Volumes
* Networks
* Docker environments

---

# 11. Create the Portainer Directory

Create the main Portainer directory:

```bash
sudo mkdir /opt/portainer
```

Create the directory used to persist Portainer data:

```bash
sudo mkdir /opt/portainer/portainer_data
```

The `portainer_data` directory stores Portainer configuration and application data.

Using persistent storage prevents the Portainer configuration from being lost when the container is recreated.

---

# 12. Create the Docker Compose File

Enter the Portainer directory:

```bash
cd /opt/portainer
```

Create the Compose file:

```bash
sudo touch docker-compose.yml
```

Verify the file:

```bash
ls -lh
```

Open the file using Nano:

```bash
sudo nano docker-compose.yml
```

Add the following configuration:

```yaml
services:
  portainer:
    container_name: portainer
    image: portainer/portainer-ce:lts
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./portainer_data:/data
    ports:
      - 9443:9443
      - 8000:8000

networks:
  default:
    name: portainer_network
```

Save the file using:

```text
Ctrl + O
Enter
Ctrl + X
```

---

# 13. Portainer Compose Configuration

The Compose configuration defines a single Portainer container.

### Container name

```yaml
container_name: portainer
```

This gives the container a predictable name.

### Portainer image

```yaml
image: portainer/portainer-ce:lts
```

The LTS image is used for the Portainer Community Edition.

### Restart policy

```yaml
restart: always
```

This allows Docker to automatically restart the container when required.

### Docker socket

```yaml
- /var/run/docker.sock:/var/run/docker.sock
```

The Docker socket allows Portainer to communicate with the Docker daemon and manage the local Docker environment.

### Persistent data

```yaml
- ./portainer_data:/data
```

This stores Portainer data outside the container filesystem.

### HTTPS interface

```yaml
- 9443:9443
```

Port `9443` is used to access the Portainer web interface.

### Portainer Agent/API port

```yaml
- 8000:8000
```

Port `8000` is also exposed according to the Compose configuration.

---

# 14. Start Portainer

From `/opt/portainer`, start the container:

```bash
sudo docker compose up -d
```

The `-d` option starts the container in detached mode.

Check the running containers:

```bash
sudo docker ps
```

The Portainer container should appear with a running status.

For example:

```text
portainer
```

with a status indicating that the container is **Up**.

---

# 15. Access Portainer

Open a browser and access:

```text
https://SERVER_IP:9443
```

Replace `SERVER_IP` with the IP address of the STB.

Portainer uses HTTPS and may initially use a self-signed certificate.

As a result, the browser may display a security warning.

Continue through the browser warning to access the Portainer setup page.

---

# 16. Portainer Initial Setup

The first Portainer page displays the **New Portainer Installation** configuration.

The setup requires:

* Username
* Password
* Password confirmation
* Setup Token

The Setup Token is required before the administrator account can be created.

---

# 17. Retrieve the Portainer Setup Token

The Setup Token can be retrieved from the Portainer container logs:

```bash
sudo docker logs portainer 2>&1 | grep -i token
```

If the token is available, the command will display it in the terminal.

Copy the token and enter it into the Portainer setup page.

The token should be treated as sensitive setup information and should not be published in a public repository.

---

# 18. Create the Portainer Administrator

Enter the required information:

* Administrator username
* Password
* Password confirmation
* Setup Token

Select **Create User**.

If the setup succeeds, Portainer will redirect to its main dashboard.

---

# 19. Portainer Setup Token Timeout

During the implementation, the Portainer setup process can encounter a timeout such as:

```text
Your Portainer instance timed out for security purposes.
```

This can happen if the initial setup page remains open for too long and the temporary Setup Token expires.

The Portainer container can be restarted:

```bash
sudo docker restart portainer
```

After restarting, retrieve a new Setup Token:

```bash
sudo docker logs portainer 2>&1 | grep -i token
```

Use the newest token displayed in the container logs and repeat the administrator setup.

---

# 20. Verify the Portainer Environment

After logging in, Portainer displays the available Docker environments.

Select **Home** to access the Docker environment.

A healthy local environment should show the Docker environment as available and running.

Portainer can then be used to inspect and manage:

* Containers
* Images
* Volumes
* Networks
* Docker resources

---

# 21. Verify the Complete Environment

At this stage, the HG680P should have the following components installed:

```text
Armbian
   │
   ├── Docker Engine
   │
   ├── Docker Compose
   │
   ├── CasaOS
   │
   └── Portainer
```

Docker provides the container runtime.

CasaOS provides a simple home-server management interface.

Portainer provides a dedicated Docker administration interface.

---

# 22. Useful Verification Commands

Check Docker:

```bash
docker --version
```

Check Docker Compose:

```bash
docker compose version
```

Check running containers:

```bash
sudo docker ps
```

Check all containers:

```bash
sudo docker ps -a
```

Check Docker images:

```bash
sudo docker images
```

Check Portainer logs:

```bash
sudo docker logs portainer
```

Check the Portainer container:

```bash
sudo docker inspect portainer
```

These commands are useful for basic troubleshooting and verification.

---

# 23. Result

After completing this stage, the HG680P has been transformed from a basic Armbian installation into a more complete Linux home-server platform.

The environment now provides:

* Armbian Linux as the operating system
* Docker as the container runtime
* Docker Compose for container configuration
* CasaOS as a web-based home-server interface
* Portainer as a Docker management interface

The server can now be used as a foundation for running additional containerized services.

---

# 24. Next Stage

The next stage of the project focuses on building a monitoring environment for the home server.

The planned monitoring stack includes:

* Prometheus
* Grafana
* Node Exporter
* cAdvisor

These components will be used to monitor system resources, Docker containers, and application performance.

---

## Related Project

This repository is the second stage of the HG680P mini-server project.

The previous stage covers:

**STB HG680P — Armbian & RTL8189FS WiFi**

The following stages continue with monitoring, automation, Docker Compose deployment, Kubernetes K3s, and GitOps using ArgoCD.

---

## Author

**Muhammad Risyad Rahmadi**

Developer Engineer

Building, automating, and deploying applications with Linux, Docker, Kubernetes, CI/CD, and GitOps.
