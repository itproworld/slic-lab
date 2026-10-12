================================================================================
HOW TO INJECT SLIC TABLES AND OEM CERTIFICATES INTO VIRTUALBOX VIRTUAL MACHINES
================================================================================

This guide provides a comprehensive, step-by-step walkthrough to activate a 
guest Windows operating system (typically Windows 7, Windows Server 2008, or 
Server 2012) using an offline OEM activation method inside Oracle VirtualBox.

For this to work, you must already have three files/pieces of data:
1. A SLIC table binary file (e.g., slic.bin)
2. A matching OEM digital certificate file (e.g., cert.xrm-ms)
3. A matching OEM:SLP product key (a 25-digit key specific to your Windows version)

---

STEP 1: COLLECT THE EXACT PATHS AND NAMES
--------------------------------------------------------------------------------
Before executing commands, you must know the exact names and directories of your 
files. For the sake of this guide, we will use the following example names:

* Virtual Machine Name: Windows7_VM
* Path to SLIC file on host: C:\OEM\slic.bin
* Path to Certificate file on host: C:\OEM\cert.xrm-ms
* Windows OEM Product Key: XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
* OEM Manufacturer Name: DELL Inc.

Note: You must adjust these placeholders to match your real files.

---

STEP 2: PREPARE YOUR HOST TERMINAL
--------------------------------------------------------------------------------
1. Completely shut down the target Virtual Machine.
2. Close the main VirtualBox manager application window so no settings get locked.
3. Open the Command Prompt or Terminal on your HOST machine:
   - On Windows: Press Win + R, type "cmd", and press Enter.
   - On Linux or macOS: Open your standard terminal application.
4. If you are on a Windows host, change your command directory to the VirtualBox 
   installation folder by running this exact command:
   
   cd "C:\Program Files\Oracle\VirtualBox"

---

STEP 3: INJECT THE SLIC BINARY FILE INTO THE VM
--------------------------------------------------------------------------------
VirtualBox allows you to pass custom ACPI tables into virtual hardware using the 
"setextradata" utility. 

Run the following command exactly as written, replacing the VM name and the file 
path with your specific details:

VBoxManage setextradata "Windows7_VM" "VBoxInternal/Devices/acpi/0/Config/CustomTable0" "C:\OEM\slic.bin"

Note: Keep the quotation marks around the VM name and the path if they contain 
any spaces. 

---

STEP 4: MATCH THE VIRTUAL BIOS STRINGS TO THE OEM MANUFACTURE
--------------------------------------------------------------------------------
Windows activation logic checks the digital markers inside the SLIC file and 
compares them against the computer's BIOS data. If the SLIC file says "DELL" 
but the VirtualBox BIOS says "VirtualBox", activation will fail.

Run the following two commands to overwrite the virtual system headers:

VBoxManage setextradata "Windows7_VM" "VBoxInternal/Devices/pcbios/0/Config/DmiBIOSVendor" "DELL Inc."

VBoxManage setextradata "Windows7_VM" "VBoxInternal/Devices/pcbios/0/Config/DmiSystemVendor" "DELL Inc."

Note: Make sure to replace "DELL Inc." with the exact manufacturer name that your 
SLIC file belongs to (e.g., "Hewlett-Packard", "LENOVO", "ASUS").

---

STEP 5: TRANSFER THE CERTIFICATE INTO THE VM GUEST OS
--------------------------------------------------------------------------------
1. Start your Virtual Machine normally.
2. Log into the guest Windows operating system.
3. Move the certificate file (cert.xrm-ms) from your host machine into the guest 
   virtual machine. You can do this by using a USB drive, setting up a Shared 
   Folder, or using VirtualBox's Drag-and-Drop feature.
4. For easy execution, copy the "cert.xrm-ms" file directly onto the root of the 
   C: drive inside the virtual machine. Its path should look like:
   C:\cert.xrm-ms

---

STEP 6: INSTALL THE CERTIFICATE AND PRODUCT KEY IN WINDOWS
--------------------------------------------------------------------------------
Now you must use the Windows Software Licensing Management tool (slmgr) to 
register the software licenses.

1. Inside the virtual machine, click the Start menu.
2. Type "cmd" into the search bar.
3. Right-click on the "Command Prompt" application icon and select "Run as Administrator". 
4. In the command prompt window, type the following command to install the 
   digital certificate:
   
   slmgr /ilc C:\cert.xrm-ms
   
   Press Enter. A window popup will appear after a few seconds confirming that 
   the certificate was successfully registered. Click OK.

5. Next, type the command to apply your matching 25-digit OEM product key:
   
   slmgr /ipk XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
   
   Replace the X's with your real key. Press Enter. Another popup will confirm 
   that the product key was installed.

---

STEP 7: CHECK FOR SUCCESSFUL ACTIVATION
--------------------------------------------------------------------------------
To ensure that all elements (SLIC table, certificate, and key) match up and 
Windows is fully activated, type the following command inside the VM:

slmgr /dli

A window will open displaying your partial product key and licensing description. 
Look closely at the line that says "License Status". If the injection worked perfectly, 
it will say "Licensed" and your system is permanently activated offline.
================================================================================
