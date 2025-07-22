# ☁️ Azure Enterprise Infrastructure – Manual Deployment Guide

This guide walks you through building a secure enterprise network on Azure using hub-and-spoke architecture, with private connectivity, Bastion access, and manual VM provisioning with IIS.

---

## 🪜 Step-by-Step Deployment

---

### ✅ Step 1: Create a Resource Group
1. Go to the [Azure Portal](https://portal.azure.com)
2. Navigate to **Resource Groups** → Click **Create**
3. Set:
   - Subscription: *Your Azure Free Subscription*
   - Resource Group Name: `EnterpriseInfraRG`
   - Region: `East US` (or closest to you)
4. Click **Review + Create** → **Create**

---

### ✅ Step 2: Create the Hub Virtual Network
1. Go to **Virtual Networks** → **Create**
2. Set:
   - Name: `VNet-Hub`
   - Address Space: `10.0.0.0/16`
3. Add a subnet:
   - Name: `AzureBastionSubnet`
   - Address Range: `10.0.1.0/24`
4. Click **Review + Create** → **Create**

---

### ✅ Step 3: Create the Spoke Virtual Network
1. Create another Virtual Network
   - Name: `VNet-Spoke`
   - Address Space: `10.1.0.0/16`
2. Add subnet:
   - Name: `WebSubnet`
   - Address range: `10.1.1.0/24`
3. Click **Review + Create** → **Create**

---

### ✅ Step 4: Peer Hub and Spoke Networks
1. Go to `VNet-Hub` → **Peerings** → Add
   - Name: `HubToSpoke`
   - Select `VNet-Spoke` as peer
   - Enable **Allow traffic** in both directions
2. Repeat from `VNet-Spoke`:
   - Name: `SpokeToHub`
   - Select `VNet-Hub` as peer

---

### ✅ Step 5: Deploy VM in Spoke VNet
1. Go to **Virtual Machines** → **Create VM**
2. Set:
   - Name: `VM-Web1`
   - Image: `Windows Server 2022`
   - Size: B1s (free-tier eligible)
   - Username/Password: *Set your own*
   - Public IP: **None**
   - VNet: `VNet-Spoke`
   - Subnet: `WebSubnet`
3. Click **Next** until **Review + Create** → **Create**

---

### ✅ Step 6: Repeat for Second VM
1. Repeat above steps for second VM:
   - Name: `VM-Web2`
   - Same settings: B1s, Windows Server 2022
   - VNet: `VNet-Spoke`
   - Subnet: `WebSubnet`
   - No public IP

---

### ✅ Step 7: Install IIS on Web VMs

After deploying `VM-Web1` and `VM-Web2`, connect to each VM using **Azure Bastion** or **RDP** and install IIS:

1. Open PowerShell as Administrator
2. Run the following command:

```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools

```



## 🔐 Step 8: Secure Blob Storage Access Using Service Endpoint

This step demonstrates how to securely connect Azure VMs to a Storage Account using a **Service Endpoint**. This ensures blob access is restricted to a specific VNet/Subnet, enhancing security while maintaining simplicity.

---

### 🛠️ 1. Create a Secure Storage Account

- Go to **Azure Portal** > **Storage Accounts** > **+ Create**
- **Settings**:
  - **Resource Group**: `RG-Hub-Spoke`
  - **Storage Account Name**: `vmblob112`
  - **Region**: Same as your VNet
- Under **Networking**:
  - Choose **"Enable access from selected virtual networks"**
  - Add:
    - **Virtual Network**: `VNet-Spoke`
    - **Subnet**: `subnet-vm` (where your VM is)
- This action **automatically adds a Service Endpoint** to the subnet.

---

### 📦 2. Upload a Test Blob

- Navigate to the created storage account > **Containers**
- Create a new container: `vmblob112-container`
  - Access level: `Private (no anonymous access)`
- Upload a sample file: `hello.txt`

---

### 💻 3. Test Blob Access from VM

From within `VM-Web1` or `VM-Web2`, open PowerShell and run:

```powershell
Invoke-WebRequest -Uri "https://vmblob112.blob.core.windows.net/vmblob112-container/hello.txt" -OutFile "C:\hello.txt"




