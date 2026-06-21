# 24/7 Persistent Ubuntu VS Code Server on Hugging Face

This guide provides a step-by-step walkthrough to deploy a persistent, secure, and fully functional **Ubuntu 24.04 LTS** cloud development environment using Hugging Face Spaces and `code-server` (VS Code in the browser), kept awake 24/7 via automated pings.

---

## 🛠️ Prerequisites

* A **Hugging Face** account.
* A **cron-job.org** account (free) to keep the environment running.

---

## 📅 Step 1: Create a Private Hugging Face Space

1. Navigate to Hugging Face and click **New Space**.
2. Give your Space a name (e.g., `VPS123`).
3. Select **Docker** as the SDK.
4. Choose the **Blank** template.
5. Set the Space visibility to **Private** to protect your workspace.

---

## 🔑 Step 2: Configure Environment Secrets

Before building the container, you must set up your secure access password.

1. Inside your Space, click on the **Settings** tab at the top right.
2. Scroll down to the **Variables and secrets** section.
3. Click **New secret**.
4. Set the key name exactly as: `PASSWORD`
5. Set the value to your preferred secure password (you will use this to log into VS Code).

---

## 🐳 Step 3: Add the Custom Ubuntu Dockerfile

Create or edit the `Dockerfile` in the root of your Space repository with the following optimized Ubuntu 24.04 configuration:

```dockerfile
FROM ubuntu:24.04

# Prevent interactive prompts during installation
ENV DEBIAN_FRONTEND=noninteractive

# Update system and install base packages + Ubuntu utilities
RUN apt-get update && apt-get install -y \
    curl \
    sudo \
    git \
    wget \
    build-essential \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# Install the latest version of code-server using the official Ubuntu script
RUN curl -fsSL https://code-server.dev/install.sh | sh

# Give the built-in 'ubuntu' user (UID 1000) passwordless sudo rights
RUN echo "ubuntu ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Prepare safe workspace directories inside the default ubuntu profile
RUN mkdir -p /home/ubuntu/workspace && chown -R ubuntu:ubuntu /home/ubuntu

USER ubuntu
WORKDIR /home/ubuntu/workspace

# Hugging Face app port
EXPOSE 7860

# Start code-server
CMD ["code-server", "--bind-addr", "0.0.0.0:7860", "--auth", "password"]
```
# Automated Keep-Alive Engine Configuration Guide (cron-job.org)

This guide details the precise configurations required to leverage the free automation tier of **cron-job.org** to keep private cloud containers, hosting environments, and Hugging Face Spaces operational 24/7.

---

## 🚀 Why This Configuration is Required

Many free-tier cloud environments (such as Hugging Face Spaces) automatically trigger container hibernation after 48 hours of public inactivity. 

Because **Private** instances completely obscure their active endpoints, typical automated HTTP pinging methods will fail or self-disable due to security handshakes. This specific configuration bypasses those blockades by utilizing standard repository polling signatures to keep the host background threads awake indefinitely.

---

## 🛠️ Step-by-Step Dashboard Setup

### 1. General Cronjob Parameters
1. Log into your dashboard at [cron-job.org](https://cron-job.org/).
2. Navigate to the **Cronjobs** tab and click the **Create cronjob** button.
3. Apply the following base values:
   * **Title:** `Hugging Face Container Uptime Engine`
   * **URL:** *Must use your central repository UI path layout (DO NOT use the direct hidden `.hf.space` domain, as private targets will throw an automatic `404 Not Found` error):*
     ```text
     [https://huggingface.co/spaces/YOUR_USERNAME/YOUR_SPACE_NAME](https://huggingface.co/spaces/YOUR_USERNAME/YOUR_SPACE_NAME)
     ```
   * **Schedule:** Select **Every 30 minutes** (or choose the user-defined option and assign a custom interval expression: `*/30 * * * *`).

---

## ⚠️ Crucial Advanced Settings (Do Not Skip)

Click the **Advanced** tab at the top of your cron job creation form interface. Because private workspaces require access tokens, unauthorized external traffic is greeted with an automatic `HTTP 401 Unauthorized` or `HTTP 3xx Redirect` response page. 

To prevent the cron system from automatically flagging your uptime ping as a dead or broken link, apply these rules:

### Rule A: Redirect Status Handling
* Locate the **Redirects** option criteria.
* **Action:** Check the setting box labeled **"Treat redirects with HTTP 3xx status code as success"**. 
* *Result:* This prevents your automated automation log graphs from filling up with false-positive warning flags.

### Rule B: Disable Failure Auto-Pause (Most Critical Step)
* Scroll down to the bottom error management options inside the Advanced container panel.
* **Action:** **TURN OFF** the toggle configuration switch stating:  
  > *"the cronjob will be disabled because of too many failures"*
* *Result:* If left active, cron-job.org will detect the `401 Unauthorized` status loop from your secure system space as a server fault and permanently disable your script automation within 3 to 5 iterations. Turning this off forces the engine to ping continuously regardless of the access token requirement.

---

## 📈 Verifying the Ping Status

Once saved, look at your cron-job.org dashboard history logs to confirm operational status:

| Expected Status Code | Meaning | Outcome |
| :--- | :--- | :--- |
| **`HTTP 200 OK`** | Direct connection successful. | **Space Stays Awake** |
| **`HTTP 302 / 303`** | Redirected to Hugging Face Login screen. | **Space Stays Awake** |
| **`HTTP 401 Unauthorized`** | Security wall hit successfully. | **Space Stays Awake** |
| **`HTTP 404 Not Found`** | **Error:** Your Space URL is typed incorrectly or misconfigured. | ❌ **Failure** (Fix URL Path) |

> 💡 **Remember:** The target container doesn't need to load your private web code to stay awake. The mere impact of incoming TCP/HTTP web traffic tracking against your Hugging Face space profile is fully sufficient to reset the idle countdown clock back to zero.
EOF
