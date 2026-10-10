
## **Setting Up a Link-OS Printer Mirror Environment Using Docker**
**Zebra-Printer: Create Mirror Server by Docker Container**

Update: 2026/10/10

![alt text](image.png)

<br/>

The Mirror feature is highly effective when you need to easily configure or update network-connected Link-OS printers. This feature is particularly valuable when pushing changes out to a large or indefinite number of printers with low cost. 

<br/>

- Configuring training and demonstration equipment
- Automated pre-deployment staging and configuration
- Automated firmware updates

<br/>

Setting up an SFTP environment to act as the Mirror server typically requires provisioning dedicated server hardware, as well as foundational server and network administration knowledge. Consequently, some users find the initial setup challenging.

To make the process more accessible, this guide provides a Docker template and a simplified, step-by-step setup procedure.

Please note that this document is intended for users who already possess basic knowledge and experience with Docker and Visual Studio Code (VS Code).
<br/>
<br/>

---

### **Table of Contents**

- [**Setting Up a Link-OS Printer Mirror Environment Using Docker**](#setting-up-a-link-os-printer-mirror-environment-using-docker)
  - [**Table of Contents**](#table-of-contents)
  - [**Method 1: Using Windows and Docker Desktop**](#method-1-using-windows-and-docker-desktop)
    - [**Prerequisites**](#prerequisites)
    - [**Step-by-Step Instructions**](#step-by-step-instructions)
  - [**Method 2: Using Debian (Linux) and Docker**](#method-2-using-debian-linux-and-docker)
    - [**Prerequisites**](#prerequisites-1)
    - [**Step-by-Step Instructions**](#step-by-step-instructions-1)

<br/>
<br/>

---

### **Method 1: Using Windows and Docker Desktop**

#### **Prerequisites**

1. Windows 11 Pro environment
2. Docker Desktop installed
3. Visual Studio Code (VS Code) installed
4. The following Docker-related VS Code extensions installed:
   1. MS Container Tools
   2. MS Docker
   3. MS Dev Containers
   4. MS Remote - SSH
   5. MS Remote Explorer

<br/>
<br/>

#### **Step-by-Step Instructions**

1. Copy the contents of the `win11_docker-desktop` directory to a location of your choice.
   <br/>
3. Navigate to the `sftp_mirror_01` directory.
   ```bash
   cd sftp_mirror_01
   ```
   <br/>

4. Start the Docker services.
   ```cmd
   sftp_mirror_01> docker compose up -d
    [+] up 2/2
    ✔ Network sftp_mirror_01_default Created                                                                           0.0s
    ✔ Container sftp_mirror_01       Started                                                                           0.4s
   ```
   <br/>

5. Verify the SFTP connection by logging in.
   - **Username:** `zebra`
   - **Password:** `zebra`

   ```cmd
   sftp_mirror_01> sftp zebra@localhost
   The authenticity of host 'localhost (::1)' can't be established.
   ED25519 key fingerprint is SHA256:+Cs31HXkVmKQnYRdvhH9prKsLr/72No705tJr8SP5a4.
   This key is not known by any other names.
   Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
   Warning: Permanently added 'localhost' (ED25519) to the list of known hosts.
   zebra@localhost's password: zebra
   Connected to localhost.
   sftp>
   ```
   <br/>

6. Configure the Mirror/SFTP settings on the printer and verify that the automated updates are working correctly.
   
   Refer to the [Zebra Programming Guide: Mirror (Page 1699)](https://docs.zebra.com/us/en/printers/software/zpl-pg/c-zpl-mirror-mirror/c-zpl-mirror-configuration/r-zpl-mirror-mirror-printer-configuration.html) for detailed configuration parameters.

<br/>
<br/>

---

### **Method 2: Using Debian (Linux) and Docker**

#### **Prerequisites**

1. Debian-based Linux (Kernel 6.12) environment
2. Docker installed
3. Visual Studio Code (VS Code) installed
4. The following Docker-related VS Code extensions installed:
   1. MS Container Tools
   2. MS Docker
   3. MS Dev Containers
   4. MS Remote - SSH
   5. MS Remote Explorer

<br/>

#### **Step-by-Step Instructions**

1. Copy the contents of the `debian` directory to a location of your choice.
2. Navigate to the `sftp_mirror_01` directory.
   ```bash
   cd sftp_mirror_01
   ```
   <br/>

3. Modify the `docker-compose.yml` file to match your specific network environment.
   ```yaml
   services:
     zebra-sftp:
       image: atmoz/sftp:latest
       container_name: sftp_mirror_01
       volumes:
         - ./zebra_mirror:/home/zebra/mirror
       command: zebra:zebra:1002:1002:mirror
       restart: unless-stopped
       networks:
         zebra_net:
           ipv4_address: 192.168.4.225 # <-- Public IP address for the Mirror server (must be distinct from the host IP)

   networks:
     zebra_net:
       driver: ipvlan # <-- Use ipvlan (macvlan is not compatible with wireless/Wi-Fi interfaces)
       driver_opts:
         parent: wlp2s0 # <-- Replace with your physical network interface name (e.g., eno1, wlan0. Check using 'ip route get 8.8.8.8')
       ipam:
         config:
           - subnet: 192.168.4.0/24 # <-- Subnet configuration
             gateway: 192.168.4.254 # <-- Gateway configuration
   ```
   
4. Start the Docker services.
   ```bash
   docker@nuc01:~/workspace/sftp-server-01$ docker compose up -d 
   [+] up 1/1
   ✔ Container sftp_mirror_01 Started   
   ```
   <br/>

5. Verify the SFTP connection by logging in.
   - **Username:** `zebra`
   - **Password:** `zebra`

   ```bash
   docker@nuc01:~/workspace/sftp-server-01$ sftp zebra@192.168.4.225 
   The authenticity of host '192.168.4.225 (192.168.4.225)' can't be established.
   ED25519 key fingerprint is SHA256:3Ze3xy8+s31UFLyLgw9lvefZ283y2EYA1Er7k9+Zpl8.
   This key is not known by any other names.
   Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
   Warning: Permanently added '192.168.4.225' (ED25519) to the list of known hosts.
   zebra@192.168.4.225's password: zebra
   Connected to 192.168.4.225.
   sftp> 
   ```
   <br/>

6. Configure the Mirror/SFTP settings on the printer and verify that the automated updates are working correctly.
   
   Refer to the [Zebra Programming Guide: Mirror (Page 1699)](https://docs.zebra.com/us/en/printers/software/zpl-pg/c-zpl-mirror-mirror/c-zpl-mirror-configuration/r-zpl-mirror-mirror-printer-configuration.html) for detailed configuration parameters.

---

**End**