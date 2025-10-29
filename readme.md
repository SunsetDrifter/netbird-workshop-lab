# NetBird Hands-On Workshop Lab

**Prerequisites:**

- Attendees bring their own laptops (Mac, Windows, or Linux).
- **Docker Desktop** installed and running.
- A "Login with..." account (Google, GitHub, Microsoft, or Email) for NetBird signup.

### **Lab 1.1: Account Setup and First Peer Enrollment**

**Step-by-Step Instructions:**

1. **Navigate to the NetBird Website:** Open a web browser and go to https://netbird.io. In the top-right corner, click the **Get Started** button.
2. **Authenticate with an Identity Provider (IdP):** On the sign-in page, select the IdP of your choice (Google, Microsoft, or GitHub). Follow the on-screen prompts to authenticate. This demonstrates NetBird's native integration with SSO providers. If MFA is enabled on the IdP account, it will be enforced automatically, providing an immediate layer of security without any extra configuration in NetBird.
3. **Download the NetBird Client:** After successful authentication, the NetBird dashboard will load, presenting an onboarding wizard. On the "Let's get your first device online" screen, click the **Install NetBird** button. An installation modal will appear; select the operating system of the laptop being used.
4. **Install the Client:** Download and run the installer, following the standard installation prompts for the operating system.
5. **Connect the Peer:** Once installed, locate the NetBird icon in the system tray (Windows/Linux) or menu bar (macOS). Click the icon and select **Connect**.
6. **Authorize the Device:** The "Connect" action will open a new browser tab, prompting for authentication again. This step securely associates the device with the user account. Authenticate using the same IdP as in Step 2. A "Login successful" message will confirm the device has been authorized.
7. **Verification:** Return to the NetBird dashboard. Navigate to the **Peers** tab in the left-hand sidebar. The participant's laptop should now be listed, showing a green "Connected" status. Note the assigned NetBird IP address (from the 100.64.0.0/10 range), which is its unique identifier on the overlay network.

### **Lab 1.2: Implementing Zero Trust Access Policies**

**Objective:** To demonstrate the core principle of ZTNA by moving from a default "allow all" network to a "default deny" posture with explicit, granular access rules.

**Step-by-Step Instructions:**

1. **Examine the Default Policy:** In the NetBird dashboard, navigate to **Access Control** -> **Policies**. Observe the policy named "Default". Note that its source is the "All" group and its destination is also the "All" group, effectively creating a full mesh network where every peer can communicate with every other peer.2
2. **Establish a "Default Deny" Posture:** Click the three-dot menu on the right side of the "Default" policy row and select **Delete**. Confirm the deletion. At this moment, the network is in a "default deny" state. If there were other peers, they would be completely unreachable.

### **Lab 2.1: Create a Reusable Setup Key**

**Objective:** To create a reusable setup key that will allow the containerized routing peer to enroll itself into the network automatically and non-interactively.

**Step-by-Step Instructions:**

1. **Navigate to Setup Keys:** In the NetBird dashboard, go to the **Setup Keys** tab in the left-hand sidebar.
2. **Create a New Key:** Click **Create Setup Key**.
3. **Configure the Key:**
    - **Name:** docker-router-key
    - **Expires in:** Select 1 day (for workshop purposes).
    - Ensure the **Reusable** checkbox is checked.
4. **Create and Copy:** Click **Create Key**. A modal will appear with the new key. Click the copy icon to copy it to your clipboard and save it in a text editor for the next lab.

### **Lab 2.2: Deploying the Simulated Private Client Network**

**Objective:** To use Docker Compose to launch a multi-container environment that simulates a client's private network, including a web server resource and a dedicated NetBird routing peer.

**Step-by-Step Instructions:**

1. **Create the Docker Compose File:** Instruct participants to open a text editor and create a new file named docker-compose.yml. They should paste the following content into the file:
    
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
    
2. **Add the Setup Key:** In the docker-compose.yml file, replace the placeholder text YOUR_SETUP_KEY_HERE with the actual setup key copied from the previous lab.
3. **Launch the Environment:** Open a terminal or command prompt, navigate to the directory where the docker-compose.yml file was saved, and execute the following command:
    
    ```bash
    docker-compose up -d
    ```
    
    This command will download the necessary images and start both containers in the background.
    
4. **Verification:**
    - In the NetBird dashboard, navigate to the **Peers** tab. A new peer should appear in the list with a "Connected" status.
    - In the terminal, find the private IP address of the Nginx server by running:
        
        ```bash
        docker inspect $(docker-compose ps -q nginx) | grep "IPAddress"
        ```
        ```Powershell
        docker inspect $(docker-compose ps -q nginx) | Select-String "IPAddress"
        ```
    - Participants should copy or write down this IP address (e.g., 172.19.0.2) for the next steps.
    - Confirm no access to the private IP address in a browser.

### **Lab 2.3: Configuring the Container-Based Routing Peer**

**Objective:** To configure the new netbird-router container peer to act as a secure gateway to the simulated Docker network.

**Step-by-Step Instructions:**

1. **Create a New Network:** In the NetBird dashboard, navigate to the **Networks** tab and click **Add Network**.
2. **Name the Network:** Give the network a descriptive name, such as Simulated Client LAN, and click **Create Network**.
3. **Define the Network Resource:** Within the new network's configuration page, under the "Resources" section, click **Add Resource**.
4. **Configure the Resource:**
    - **Name:** Nginx Server
    - **Address:** Enter the private IP address of the nginx-server container identified in the previous lab, followed by a /32 CIDR suffix (e.g., 172.19.0.2/32).
    - **Assign to Group:** In the "Assign to a Destination Group" field, select the Workshop-Resources group.
    - Click **Add Resource**.
5. **Assign a Routing Peer:** Under the "Routing Peers" section, click **Add Routing Peer**. A list of available peers will appear. Select the netbird-router peer (Docker container peer).
6. **Enable Masquerading:** In the configuration pane for the routing peer, ensure the **Masquerade** toggle is enabled. This is a critical step that makes all traffic appear to come from the routing peer's own IP address within the private network, solving routing issues.
7. **Save the Configuration:** Click **Add Routing Peer** to save the settings.

### **Lab 2.4: Defining Access and Validating the Connection**

**Objective:** To create a specific ZTNA policy that grants the admin group access to the newly defined network resource, and then to test the end-to-end connectivity.

**Step-by-Step Instructions:**

1. **Navigate to Policies:** In the dashboard, go to **Access Control** -> **Policies** and click **Add Policy**.
2. **Create the Access Policy:** Configure the new policy as follows:
    - **Name:** Admins to Nginx
    - **Source Groups:** Select Workshop-Admins.
    - **Destination Groups:** Select Workshop-Resources.
    - **Protocol:** Select "TCP".
    - **Ports:** Enter 80.
    - **Flow:** Set the flow to unidirectional (->). This enforces least privilege.
3. **Save the Policy:** Click **Add Policy**.
4. **Final Verification:** Instruct participants to open a new browser tab and navigate directly to the private IP address of their nginx-server container (e.g., http://172.19.0.2).
5. **The Result:** The "Welcome to nginx!" page should load successfully.

---
