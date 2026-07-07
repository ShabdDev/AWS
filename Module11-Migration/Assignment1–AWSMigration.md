# Module 11: Assignment 1 - On-Premises Environment Preparation for AWS Cloud Migration

## Problem Statement
XYZ Corporation is planning to migrate its structural server workloads onto AWS Cloud infrastructure to maximize continuous service availability and network execution speeds. As a pre-migration requirement, system engineers must build and structure a representative on-premises source database/web server locally. Implement this target baseline setup by installing an operational Ubuntu Server Virtual Machine (VM) using Oracle VirtualBox hypervisor systems.

---

## Tasks To Be Performed
1. Download and install Oracle VirtualBox on your local host system workstation.
2. Download the target Ubuntu Server ISO binary image disk profile.
3. Configure and deploy a fully operational standalone Ubuntu Server VM on your local workstation machine to act as the migration source environment.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Install the Hypervisor Environment (Oracle VirtualBox)
1. Open your web browser and navigate to the official download portal: `https://www.virtualbox.org/wiki/Downloads`.
2. Select the binary download package link corresponding to your host computer architecture (e.g., **Windows hosts** or **macOS / Linux distributions**).
3. Once the installer package completes downloading, double-click the file to launch the wizard instructions.
4. Advance through the configuration settings by clicking **Next** on all prompts, keeping standard storage targets active, and finalizing with **Install**.

---

### Step 2: Download the Source Operating System ISO Image
1. Click or reference the official tracking network URL distribution mirror link provided: `https://releases.ubuntu.com/18.04.5/ubuntu-18.04.5-live-server-amd64.iso`.
2. Save the bootable file package directly onto your local machine workspace (e.g., inside your default `Downloads` or `Desktop` file directory).

---

### Step 3: Provision and Spin Up the Ubuntu Server Virtual Machine
1. Launch the **Oracle VirtualBox Manager** software console application on your desktop screen.
2. Click on the blue **New** button (Sparks icon) located in the upper functional tools toolbar.
3. Configure the **Virtual Machine Identity Settings**:
   * **Name:** `XYZ-OnPrem-Source-Server`
   * **Type:** Select **Linux**.
   * **Version:** Select **Ubuntu (64-bit)**.
   * Click **Next**.
4. **Memory Allocation:** Assign a minimum threshold of **`1024 MB`** or **`2048 MB`** (1GB or 2GB RAM based on local host hardware availability constraints) and click **Next**.
5. **Virtual Hard Disk:** Select **Create a virtual hard disk now** > Choose **VDI (VirtualBox Disk Image)** > Select **Dynamically allocated**. Set storage capacity bounds to **`20 GB`** and click **Create**.
6. **Mount the Installation ISO Image (Boot Cycle Setup):**
   * Select your newly generated virtual container item: `XYZ-OnPrem-Source-Server`.
   * Click on the yellow **Settings** gear cog icon in the top toolbar row.
   * Go to the **Storage** category panel from the left indexing directory list.
   * Under the *Storage Devices* hierarchy, highlight the label reading **Empty** positioned directly beneath the *Controller: IDE* entry.
   * Locate the disk icon shortcut item on the far right under attributes, click **Choose a disk file**, and select the downloaded **`ubuntu-18.04.5-live-server-amd64.iso`** payload file. Click **OK**.
7. **Boot the Operating System:**
   * Highlight your configured virtual core element and click the green **Start** arrow button.
   * A terminal emulation window will launch. Follow the standard on-screen text instructions: select language, skip default network proxy blocks, authorize storage partitioning setups, input username/password credentials (e.g., `adminuser` / `SecurePass123`), and allow the installation script loop to execute fully.
   * Once complete, execute a virtual system reboot inside the interface panel. The system will cleanly present a terminal terminal command login prompt, verifying a successful mock on-premises baseline staging setup.
  
---

## Part 2: Step-by-Step Deletion Process (Clean-up)

Since this virtualization setup runs entirely inside your personal machine computing resources, you can erase this container setup cleanly at any time to recover native hardware capacity limits:

### 1. Wipe the Virtual Machine from the Hypervisor Dashboard

1. Inside the **Oracle VirtualBox Manager** utility window interface, highlight the virtual machine labeled **`XYZ-OnPrem-Source-Server`**.
2. Right-click the element row target item and select **Remove** from the dropdown option settings menu list.
3. A verification prompt window will appear. Click on the button labeled **Delete all files**. This guarantees that all nested disk image allocations (`.vdi` blocks) are securely erased from your computer storage space.

### 2. Remove Downloaded ISO Media Archives

1. Open your host filesystem directory tool (e.g., Windows File Explorer or Mac Finder).
2. Browse to the directory containing the downloaded file package asset: `ubuntu-18.04.5-live-server-amd64.iso`.
3. Select the data component asset, execute a permanent deletion command sequence, and clear your Recycle Bin folders to finish storage maintenance.
