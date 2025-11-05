# NetBird Hands-On Workshop Lab

## Prerequisites

- Laptop (Mac, Windows, or Linux).
- Terminal for SSH
- Google, GitHub, Microsoft, or Email account for NetBird signup.

## Lab 1.1: Account Setup and First Peer Enrollment (Laptop)

1. Go to https://netbird.io and click **Get Started**
2. Authenticate with your IdP (Google, Microsoft, or GitHub)
3. Click [Install NetBird](https://app.netbird.io/install) and select your operating system
4. Download and install the client
5. Click the NetBird icon in your system tray/menu bar and select **Connect**
6. Authenticate again in the browser to authorize your device
7. Verify your laptop appears in the **Peers** tab with a "Connected" status and a NetBird IP address
8. Navigate to Team - Users and add your user to a group called 'Admins'

## Lab 1.2: Implementing Zero Trust Access Policies

**Objective:** Move from default "allow all" to "default deny" with explicit access rules.

1. Test NGINX working in browser BEFORE disabling policy. http://10.39.96.3
2. **Ping** the NGINX server, then disable that policy while ping is active
3. Navigate to **Access Control** -> **Policies** and observe the "Default" policy (All → All)
4. Delete the "Default" policy to establish a "default deny" posture

## Lab 2.1: Create a Reusable Setup Key

1. Go to **Setup Keys** tab and click **Create Setup Key**
2. Configure:
   - **Name:** vps-router-key
   - **Expires in:** 1 day
   - Enable **Reusable**
3. Click **Create Key**, copy it, and save for the next lab

## Lab 2.2: Set Up the Routing Peer in the Virtual Private Cloud

1. SSH into the public IP address on the index card.
2. Install NetBird: curl -fsSL https://pkgs.netbird.io/install.sh | sh
3. netbird up --setup-key YOUR_SETUP_KEY_HERE
4. Replace `YOUR_SETUP_KEY_HERE` with your actual setup key
5. Enter and confirm the routing peer is listed in the Peers view.

## Lab 2.3: Configuring the NetBird Network

1. Navigate to **Networks** tab and click **Add Network**
2. Name it (e.g., "VPS Network") and click **Create Network**
3. Under "Resources", click **Add Resource**:
   - **Name:** Nginx Server
   - **Address:** 10.39.96.3/32
   - **Assign to Group:** VPS-Resources
   - Click **Add Resource**
4. Under "Routing Peers", click **Add Routing Peer** and select the peer you installed in the VPS
6. Click **Add Routing Peer** to save

## Lab 2.4: Defining Access and Validating the Connection

1. Navigate to **Access Control** -> **Policies** and click **Add Policy**
2. Configure the policy:
   - **Name:** Admins to Nginx
   - **Source Groups:** Admins
   - **Destination Groups:** VPS-Resources
   - **Protocol:** TCP
   - **Ports:** 80
   - **Flow:** Unidirectional (->)
3. Click **Add Policy**
4. Open your browser and navigate to your nginx container's IP: **http://10.39.96.3**
5. You should see the "Welcome to nginx!" page

---
