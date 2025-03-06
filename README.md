# Recovering-Data-and-Restoring-Windows-from-a-VHDX-Backup
This guide outlines the steps to recover data from a backup disk in VHDX format and restore a Windows system (e.g., Windows Server Active Directory) using the VHDX file. The process involves mounting the VHDX and configuring the Boot Configuration Data (BCD) store using bcdedit to boot from the VHDX.
Tested Date: March 06, 2025

Author: [Dereje S.]

Environment: Windows (assumed Windows Server for AD restoration)

## Prerequisites
- A VHDX file containing a backup of the Windows system (e.g., C:\path\to\your.vhdx).
- Administrative access to a Windows machine with bcdedit available.
- The VHDX file must be accessible on the local disk (e.g., C:).
- Ensure the system supports booting from a VHDX (Hyper-V or native VHD boot must be enabled).

## Steps
### 1. Verify the VHDX File
- Confirm the VHDX file exists at the specified location (e.g., C:\path\to\your.vhdx).
- Ensure the file is not corrupted by mounting it manually:
1. Open Disk Management (diskmgmt.msc).
2. Click Action > Attach VHD.
3. Browse to the VHDX file and mount it.
4. Verify the data (e.g., Windows system files, Active Directory data) is intact.
5. Detach the VHDX after verification.

### 2. Open an Elevated Command Prompt
- Press Win + X and select Command Prompt (Admin) or Windows PowerShell (Admin).
- Alternatively, search for cmd, right-click, and select Run as administrator.

### 3. Create a New BCD Entry
- Run the following command to duplicate the current boot entry and create a new one labeled "Windows Server AD":

`bcdedit /copy {current} /d "Windows Server AD"`

- This command outputs a GUID (e.g., {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}). Note this GUID, as it identifies the new boot entry. Replace {GUID} in subsequent steps with this value.
## 4. Configure the VHDX as the Boot Device
- Set the device and OS device to the VHDX file using the following commands, replacing {GUID} with the value from Step 3 and C:\path\to\your.vhdx with the actual path to your VHDX file:

`bcdedit /set {GUID} device vhd=[C:]\path\to\your.vhdx`
`bcdedit /set {GUID} osdevice vhd=[C:]\path\to\your.vhdx`
- Note: The [C:] syntax (with square brackets) specifies the drive letter where the VHDX resides. Ensure there are no extra spaces or slashes.

## 5. Enable HAL Detection
- Configure the boot entry to detect the Hardware Abstraction Layer (HAL) automatically:

`bcdedit /set {GUID} detecthal on`
- This ensures compatibility with the underlying hardware when booting from the VHDX.
## 6. Verify the BCD Configuration
- Run the following command to review the BCD store:

  `bcdedit /enum`
- Look for the "Windows Server AD" entry and confirm:

   - The device and osdevice fields point to vhd=[C:]\path\to\your.vhdx.
   - The detecthal option is set to yes.
## 7. Restart and Test the Boot
- Reboot the system:
`shutdown /r /t 0`

- During boot, select the "Windows Server AD" option from the boot menu.
- Verify that the system boots successfully from the VHDX and that all data (e.g., 
   Active Directory) is intact.

# Example Commands
Assuming the VHDX file is located at C:\Backups\ADBackup.vhdx and the GUID generated is {a1b2c3d4-5678-9012-3456-7890abcdef12}, the full sequence would be:

`bcdedit /copy {current} /d "Windows Server AD"`

`bcdedit /set {a1b2c3d4-5678-9012-3456-7890abcdef12} device vhd=[C:]\Backups\ADBackup.vhdx`

`bcdedit /set {a1b2c3d4-5678-9012-3456-7890abcdef12} osdevice vhd=[C:]\Backups\ADBackup.vhdx`

`bcdedit /set {a1b2c3d4-5678-9012-3456-7890abcdef12} detecthal on`

# Troubleshooting
- Boot Failure: If the system doesn’t boot, recheck the GUID and VHDX path in the BCD store.
- "Element not found" Error: Ensure the VHDX path is correct and the file is accessible.
- Missing Boot Option: Use `bcdedit /displayorder {GUID}` to add the entry to the boot menu.
# Success Confirmation
This procedure was successfully tested on March 06, 2025. The system booted from the VHDX, and the restored Windows Server AD environment functioned as expected.

# Additional Notes
To make this the default boot option, run:

'bcdedit /default {GUID}`

- To remove the entry if no longer needed:

`bcdedit /delete {GUID}`

#### THANKS
