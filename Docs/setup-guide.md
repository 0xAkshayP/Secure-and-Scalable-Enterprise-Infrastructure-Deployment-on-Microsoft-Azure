# ☁️ Azure Secure and Scalable Enterprise Infrastructure – Manual Deployment Guide

This guide walks you through building a secure enterprise network on Azure using hub-and-spoke architecture, with private connectivity, Bastion access, and manual VM provisioning with IIS.

---

## 🪜 Step-by-Step Deployment

---

### ✅ Step 1: Create a Resource Group
1. Go to the [Azure Portal](https://portal.azure.com)
2. Navigate to **Resource Groups** → Click **Create**
3. Set:
   - Subscription: *Your Azure Free Subscription*
   - Resource Group Name: `Azure-Project1`
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
  - **Resource Group**: `Azure-Project1`
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

```

✅ If successful, the file hello.txt is downloaded to the VM.

❌ Accessing this file from a public IP or browser should result in Access Denied.

## Step 9: Configure Azure Private Endpoint and DNS for Blob Storage

To securely access Azure Blob Storage from a VM using a private IP, we configure a **Private Endpoint** with DNS integration.

---

### ✅ Objective

- Create a Private Endpoint for the storage account (`vmblob221`)
- Integrate with a Private DNS zone to resolve blob storage via private IP
- Verify secure internal access

---

### 🔧 Tasks Performed

1. **Created a Private Endpoint** for the storage account:
   - Resource: `vmblob221`
   - Sub-resource type: `blob`
   - VNet/Subnet: `Spoke VNet` used by the VM
   - Private IP assigned: e.g., `10.0.1.x`

2. **Linked Private DNS Zone**:
   - Zone: `privatelink.blob.core.windows.net`
   - Linked to the `Spoke VNet` where VM is deployed
   - Auto-registration enabled ✅

3. **Verified DNS Resolution** from the VM:

   Ran in PowerShell inside the VM:

   ```powershell
   nslookup vmblob221.blob.core.windows.net
   ```
   
## 🔟 Step 10: Configure Azure Monitor Alerts for VM and Resource Monitoring

In this step, we configure an Azure Monitor Alert to notify us when our virtual machine (`VM-Web2`) exceeds a defined CPU usage threshold.

---

### 🎯 Objective:
Trigger an alert when CPU usage is greater than 80% for a continuous duration of 5 minutes.

---

### ✅ Steps:

1. **Open Azure Monitor**
   - Go to the Azure portal and search for **"Monitor"** in the top search bar.
   - Click on the Monitor service.

   ![Monitor Overview](images/monitor-overview.png)

2. **Create a New Alert Rule**
   - Inside Monitor, go to **Alerts** from the sidebar.
   - Click **+ Create** → **Alert rule**.

3. **Define the Scope**
   - Under **Scope**, click **Select resource**.
   - Choose your virtual machine `VM-Web2`.

   ![Select VM Scope](images/select-vm-scope.png)

4. **Add a Condition**
   - Click on **Add condition**.
   - Select **Signal name:** `Percentage CPU`.
   - Configure:
     - **Operator:** Greater than  
     - **Threshold value:** 80  
     - **Aggregation type:** Average  
     - **Over the last:** 5 minutes

5. **Create an Action Group**
   - Click **Add action group**.
   - Set a name, such as `Email-Notify-VM`.
   - Add your email address for notifications under **Email/SMS/Push/Voice**.

6. **Name and Create the Alert Rule**
   - Name the alert rule: `HighCPU-Alert-VM-Web2`
   - Set Severity to **3 (Informational)** or as appropriate.
   - Click **Review + Create** and then **Create**.

   ![Alert Summary](images/alert-summary.png)

---

### 📌 Summary:
You’ve now configured an Azure Monitor alert rule to track CPU usage on your VM. This helps you proactively monitor and respond to performance issues.




