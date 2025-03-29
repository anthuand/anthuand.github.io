# Tutorial: Installing Open WebUI on Raspberry Pi 5 and Connecting to Groq with Remote Access via Twingate

![Raspberry Pi Setup](https://images.unsplash.com/photo-1552283576-3ea3519bf12e?w=600&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8N3x8cmFzcGJlcnJ5JTIwcGl8ZW58MHx8MHx8fDA%3D)

## Introduction

This tutorial guides you through installing and configuring **Open WebUI** on a Raspberry Pi 5, integrating it with Groq's API for AI capabilities, and setting up secure remote access using Twingate. With this configuration, you can leverage Groq's AI models and manage your Raspberry Pi from anywhere.

---

## Prerequisites

Before you begin, ensure you have:

- **Raspberry Pi 5** with at least 4GB of RAM  
- **Operating System**: Raspberry Pi OS (64-bit), fully updated  
- **Docker** installed and running  
- A **Groq Cloud** account  
- A **Twingate** account  


> **Note**: A stable internet connection is required to download Docker images and configurations.

---

## Step 1: Prepare the Raspberry Pi

### 1.1 Update the Operating System

Update your package list and upgrade the system:

```bash
sudo apt update
sudo apt upgrade -y
```

### 1.2 Install Docker (if not already installed)

Follow one of these guides to install Docker on your Raspberry Pi:

- [Pimylifeup: Docker on Raspberry Pi](https://pimylifeup.com/raspberry-pi-docker/)
- [Official Docker Documentation](https://docs.docker.com/engine/install/debian/)

> **Tip**: To avoid using `sudo` with every Docker command, add your user to the `docker` group:
>
> ```bash
> sudo usermod -aG docker $USER
> ```
>
> Then, log out and back in for the changes to take effect.

### 1.3 Install Docker Compose (Optional but Recommended)

Docker Compose simplifies container management. Install it with:

```bash
sudo apt install docker-compose -y
```

---

## Step 2: Set Up Open WebUI

### 2.1 Create a Directory for Open WebUI

Create a directory to store the Open WebUI files:

```bash
sudo mkdir -p /opt/stacks/openwebui
cd /opt/stacks/openwebui
```

### 2.2 Create the `docker-compose.yml` File


Inside this directory, create a file named `docker-compose.yml` using the following command:

```bash
sudo nano docker-compose.yml
```

Then, add the following content:

```yaml
version: '3'
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    volumes:
      - ./data:/app/backend/data
    ports:
      - "3000:8080"
    extra_hosts:
      - host.docker.internal:host-gateway
    restart: unless-stopped
```

**Details:**

- **image**: Specifies the official Open WebUI image.
- **volumes**: Maps the local `./data` directory to the container for persistent storage.
- **ports**: Exposes port `3000` on your Raspberry Pi (mapped to port `8080` in the container).
- **restart**: Ensures the container restarts automatically if the Raspberry Pi reboots or if the container stops unexpectedly.

### 2.3 Launch Open WebUI

Start the service by running:

```bash
docker-compose up -d
```

> **Verification**:  
> Check that the service is running:
> ```bash
> docker-compose ps
> ```
> And view logs with:
> ```bash
> docker-compose logs -f
> ```

---

## Step 3: Connect to Groq via API

![Artificial Intelligence](../statics/imgs/ai.png)

### 3.1 Create a Groq Cloud Account

If you don't have an account yet, sign up at [Groq Cloud](https://console.groq.com) and complete the registration process.

### 3.2 Generate an API Key

In your Groq dashboard:

1. Navigate to **API Keys**.
2. Create a new key (e.g., "Open WebUI").
3. Copy the generated **API Key**.

### 3.3 Configure the Connection in Open WebUI

1. Open your web browser and navigate to `http://<YOUR-PI-IP>:3000`.
2. Click your username or profile icon in the lower-left corner and select **Settings**.
3. In the **Connections** tab, click the **+** button.
4. Enter the Groq base URL:
   ```
   https://api.groq.com/openai/v1
   ```
5. Paste your Groq **API Key**.
6. Save your changes.

Now, Open WebUI will communicate with Groq’s API to process AI queries.

---

## Step 4: Set Up Twingate for Remote Access

![Network Security](https://images.unsplash.com/photo-1603985529862-9e12198c9a60?w=600&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8Mnx8dnBufGVufDB8fDB8fHww)

Twingate provides a Zero Trust solution for secure remote access, ensuring that your Raspberry Pi and Open WebUI are accessible only to authorized users. This section details the complete setup process.

### 4.1 Create a Twingate Account

- Visit [Twingate](https://www.twingate.com/) and sign up for an account.
- After registration, log in to access the **Admin Dashboard** where you can manage connectors, resources, and user permissions.

### 4.2 Install the Twingate Connector on Your Raspberry Pi

1. **Download the Connector:**
   - From your Twingate Admin Dashboard, navigate to the **Connectors** section.
   - Download the Linux connector package suitable for Debian/Raspberry Pi OS.
2. **Install the Connector:**
   - Follow Twingate’s installation instructions. Typically, this involves running a command such as:
     ```bash
     sudo dpkg -i twingate-connector-package.deb
     ```
   - Once installed, the connector will register your Raspberry Pi with Twingate as a secure gateway.
3. **Verify Installation:**
   - Check the connector status from the Twingate Admin Dashboard. It should show as active and connected.

### 4.3 Configure a Resource for Open WebUI

1. **Add a New Resource:**
   - In the Twingate Admin Dashboard, click **Add Resource**.
2. **Enter Resource Details:**
   - **Resource Name:** e.g., `Open WebUI`
   - **Address:** The IP address or hostname of your Raspberry Pi (e.g., `192.168.1.50`)
   - **Port:** `3000` (as configured in the docker-compose file)
3. **Assign Permissions:**
   - Link the resource to a specific group or user who needs access.
   - Configure policies if necessary, such as limiting access to certain time frames or IP ranges.
4. **Save and Apply:**
   - Confirm and save your configuration. The new resource should now appear in your resource list.

### 4.4 Connect Remotely with Twingate

1. **Install the Twingate Client:**
   - On your computer or mobile device, download and install the Twingate application.
2. **Establish a Connection:**
   - Launch the Twingate client and log in using your Twingate account credentials.
   - Select your configured network, and the client will automatically connect to your private network.
3. **Access Open WebUI:**
   - Open a web browser and navigate to the resource URL (e.g., `http://192.168.1.50:3000` or the custom name you assigned).
   - You should now see the Open WebUI interface, accessible securely from anywhere.

> **Additional Tips:**
>
> - **Monitor Activity:** Regularly check the Twingate Admin Dashboard to monitor connector status and access logs.
> - **Security Policies:** Consider enabling multi-factor authentication (MFA) on your Twingate account for added security.
> - **Connector Updates:** Keep the Twingate Connector up-to-date by following notifications from Twingate for any available updates or patches.

---

## Conclusion

Congratulations! You have successfully:

1. Installed and configured Open WebUI on your Raspberry Pi 5.
2. Integrated Open WebUI with Groq’s API for advanced AI capabilities.
3. Set up Twingate to secure and simplify remote access to your Raspberry Pi.

Enjoy your new AI environment with secure remote connectivity!

---

## References

- [Pimylifeup: Docker on Raspberry Pi](https://pimylifeup.com/raspberry-pi-docker/)
- [Official Docker Documentation](https://docs.docker.com/engine/install/debian/)
- [Groq Cloud](https://console.groq.com)
- [Twingate](https://www.twingate.com/)
- [Open WebUI GitHub Repository](https://github.com/open-webui/open-webui)

> **Note**: Always safeguard your API keys and review the usage policies of each service to avoid security or billing issues.

---

*Images provided by [Unsplash](https://unsplash.com/) under the Unsplash License.*

