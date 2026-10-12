# Step-by-Step Guide: How to Use SLIC in VirtualBox

Configuring a **SLIC (Software Licensing Description Table)** in **VirtualBox** allows you to use your physical computer's OEM factory activation inside a Virtual Machine (VM). 

---

## 💡 How OEM Offline Activation Works
OEM activation relies on a **three-way cryptographic match**:
1. **The SLIC Table:** A slice of data embedded directly into your physical motherboard's BIOS/UEFI.
2. **The OEM Certificate (`.xrm-ms` file):** A digital certificate matching the signature inside the BIOS SLIC table.
3. **The OEM SLP Product Key:** A generic master product key used by the manufacturer (e.g., Dell, Lenovo) for that specific edition of Windows.

When all three align perfectly, Windows activates completely offline without contacting Microsoft servers.

---

## 🛠️ Step 1: Extract the SLIC Table from your Physical PC (Host)
You must grab the raw binary data of the SLIC table from your real computer's motherboard.

### If your Host OS is Windows:
1. Download a free hardware utility like **RWEverything** (Read & Write Everything).
2. Open the application and click on the **ACPI** button at the top toolbar.
3. Click on the **SLIC** tab.
4. Click **Access** and then select **Save Image**.
5. Save the file exactly as `slic.bin`.

### If your Host OS is Linux:
Open your terminal and run the following command to dump the binary table:
```bash
sudo acpidump -t SLIC -o slic.bin -b
```

---

## 🛠️ Step 2: Bind the SLIC File to the Virtual Machine
Now, instruct VirtualBox to inject this file into the virtual motherboard of your VM.

1. Close the VirtualBox user interface entirely.
2. Open your Host's terminal (or **Command Prompt / PowerShell** as an Administrator if on Windows).
3. Navigate to your VirtualBox directory (usually `C:\Program Files\Oracle\VirtualBox` on Windows).
4. Run the following command:

```bash
vboxmanage setextradata "Your_VM_Name" "VBoxInternal/Devices/acpi/0/Config/CustomTable0" "/path/to/slic.bin"
```

⚠️ **Crucial Rules:**
* Replace `"Your_VM_Name"` with the exact name of your virtual machine in quotes.
* Replace `"/path/to/slic.bin"` with the exact, absolute path to where you saved the file in Step 1 (e.g., `"C:\VMs\slic.bin"`).
* **Case Sensitivity Matters:** `CustomTable0` must be capitalized exactly as shown.

---

## 🛠️ Step 3: Inject Matching DMI BIOS Strings
Sometimes Windows checks if the motherboard text strings match the SLIC vendor. Run these commands to align the virtual BIOS information with your hardware manufacturer:

```bash
vboxmanage setextradata "Your_VM_Name" "VBoxInternal/Devices/pcbios/0/Config/DmiBIOSVendor" "MANUFACTURER_NAME"
vboxmanage setextradata "Your_VM_Name" "VBoxInternal/Devices/pcbios/0/Config/DmiSystemVendor" "MANUFACTURER_NAME"
```
*(Replace `MANUFACTURER_NAME` with the actual brand of your physical PC, such as `Dell`, `Hewlett-Packard`, `LENOVO`, or `ASUS`)*.

---

## 🛠️ Step 4: Inject the Certificate and Key (Inside the VM)
Now, turn on your Virtual Machine. The VM contains the SLIC table, but the guest Windows OS still needs the certificate and the key.

### 1. Install the OEM Certificate
1. Obtain the official `.xrm-ms` digital certificate file matching your physical PC's brand (e.g., `dell.xrm-ms` or `lenovo.xrm-ms`).
2. Move the certificate file inside your running Virtual Machine (e.g., paste it into `C:\cert\`).
3. Inside the VM, search the Start Menu for `cmd`, right-click **Command Prompt**, and select **Run as Administrator**.
4. Run the following command to inject the certificate:
   ```cmd
   slmgr /ilc C:\cert\your_manufacturer_cert.xrm-ms
   ```
   *If successful, a popup window will say: "License file installed successfully."*

### 2. Install the Matching OEM SLP Key
You must use the generic, factory master key assigned to your manufacturer for that specific edition of Windows (do not use the unique key printed on a sticker on your computer case).

In the same administrative Command Prompt, run:
```cmd
slmgr /ipk XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
```
*(Replace the X's with the generic OEM-SLP master key corresponding to your Windows version and brand).*

### 3. Trigger Offline Validation
Force Windows to evaluate the three-way cryptographic match right away by running:
```cmd
slmgr /ato
```
Windows will instantly verify the key against the certificate, trace it to the virtualized SLIC table, and change its status to **Activated**.
