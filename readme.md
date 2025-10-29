# NetBird Hands-On Workshop Lab

## Prerequisites

- Laptop (Mac, Windows, or Linux).
- Docker installed and running.
- Google, GitHub, Microsoft, or Email account for NetBird signup.

## Lab 1.1: Account Setup and First Peer Enrollment

1. Go to https://netbird.io and click **Get Started**
2. Authenticate with your IdP (Google, Microsoft, or GitHub)
3. Click [Install NetBird](https://app.netbird.io/install) and select your operating system
4. Download and install the client
5. Click the NetBird icon in your system tray/menu bar and select **Connect**
6. Authenticate again in the browser to authorize your device
7. Verify your laptop appears in the **Peers** tab with a "Connected" status and a NetBird IP address

## Lab 1.2: Implementing Zero Trust Access Policies

**Objective:** Move from default "allow all" to "default deny" with explicit access rules.

1. Navigate to **Access Control** -> **Policies** and observe the "Default" policy (All → All)
2. Delete the "Default" policy to establish a "default deny" posture

## Lab 2.1: Create a Reusable Setup Key

1. Go to **Setup Keys** tab and click **Create Setup Key**
2. Configure:
   - **Name:** docker-router-key
   - **Expires in:** 1 day
   - Enable **Reusable**
3. Click **Create Key**, copy it, and save for the next lab

## Lab 2.2: Deploying the Simulated Private Client Network

1. Create `docker-compose.yml` with the following content:
    
    YAML
    
    ```bash
    version: "3.7"
    services:
      nginx:
        image: nginx:latest
        container_name: nginx-server
        # No ports needed here, we will access it over the private network
    
      netbird-router:
        image: netbirdio/netbird:latest
        container_name: netbird-router
        cap_add:
          - NET_ADMIN
          - SYS_ADMIN
          - SYS_RESOURCE
        environment:
          # Replace the key below with the one you just created
          - NB_SETUP_KEY=YOUR_SETUP_KEY_HERE
        volumes:
          - netbird-client:/var/lib/netbird
    
    volumes:
      netbird-client:
    ```
    
2. Replace `YOUR_SETUP_KEY_HERE` with your actual setup key
3. Launch the environment:
    
    ```bash
    docker-compose up -d
    ```
    
4. Verify:
   - Check **Peers** tab for new "Connected" peer
   - Get the Nginx container's private IP:
     ```bash
     # Mac/Linux
     docker inspect $(docker-compose ps -q nginx) | grep "IPAddress"
     
     # Windows PowerShell
     docker inspect $(docker-compose ps -q nginx) | Select-String "IPAddress"
     
     # Windows CMD
     docker inspect nginx-server | findstr "IPAddress"
     ```
   - Note the IP address (e.g., 172.18.0.2)
   - Confirm the IP is not accessible in your browser yet

## Lab 2.3: Configuring the Container-Based Routing Peer

1. Navigate to **Networks** tab and click **Add Network**
2. Name it (e.g., "Simulated Client LAN") and click **Create Network**
3. Under "Resources", click **Add Resource**:
   - **Name:** Nginx Server
   - **Address:** Your container's IP with /32 (e.g., 172.18.0.2/32)
   - **Assign to Group:** Workshop-Resources
   - Click **Add Resource**
4. Under "Routing Peers", click **Add Routing Peer** and select the netbird-router peer
5. Enable the **Masquerade** toggle
6. Click **Add Routing Peer** to save

## Lab 2.4: Defining Access and Validating the Connection

1. Navigate to **Access Control** -> **Policies** and click **Add Policy**
2. Configure the policy:
   - **Name:** Admins to Nginx
   - **Source Groups:** Workshop-Admins
   - **Destination Groups:** Workshop-Resources
   - **Protocol:** TCP
   - **Ports:** 80
   - **Flow:** Unidirectional (->)
3. Click **Add Policy**
4. Open your browser and navigate to your nginx container's IP (e.g., http://172.18.0.2)
5. You should see the "Welcome to nginx!" page

---
