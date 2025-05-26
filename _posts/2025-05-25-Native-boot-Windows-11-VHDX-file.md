---
title: "Native Boot Windows 11 from a VHDX File"
date: 2025-05-25 23:00:00 +0000
categories: [Windows, Boot]
tags: [windows, boot, winpe]
---

# Native Boot Windows 11 from a VHDX File: Comprehensive Guide #

Native booting Windows 11 from a VHDX file enables you to run a full Windows installation directly from a virtual hard disk without virtualization overhead. This approach is particularly useful when adding Windows to an existing Linux system while maintaining separation between operating systems.

## Prerequisites ##

- Windows 11 ISO file
- Existing Linux PC
- One logical or physical drive with 50GB+ free NTFS-formatted space
- WinPE USB boot key or Ventoy with WinPE ISO
- Basic familiarity with command line operations

## Step 1: Boot into WinPE and Prepare the VHDX File ##

1. **Boot from your WinPE media**:
    - Insert your WinPE USB or boot from Ventoy with WinPE ISO
    - Restart your computer and boot from the media

2. **Open Command Prompt in WinPE**:
    - Press `Shift+F10` or find Command Prompt in WinPE interface

3. **Identify your NTFS drive**:
    ```cmd
    diskpart
    list disk
    list volume
    ```
    Note the drive letter of your NTFS-formatted volume (e.g., D:)

4. **Create a VHDX file using diskpart**:
    ```cmd
    diskpart
    create vdisk file="D:\VirtualDisks\Win11.vhdx" maximum=50000 type=fixed
    select vdisk file="D:\VirtualDisks\Win11.vhdx"
    attach vdisk
    create partition primary
    format quick fs=ntfs label="Win11Boot"
    assign letter=V
    exit
    ```
    (Adjust path and size as needed; V: will be the letter assigned to your VHDX)

## Step 2: Deploy Windows 11 to the VHDX ##

1. **Mount your Windows 11 ISO in WinPE**:
    ```cmd
    dism /mount-image /imagefile:D:\path\to\Windows11.iso /mountdir:E:\ /index:1
    ```
    (If this doesn't work, manually copy ISO to NTFS drive and right-click → Mount)

2. **Apply the Windows image**:
    ```cmd
    dism /Apply-Image /ImageFile:"E:\sources\install.wim" /Index:1 /ApplyDir:"V:\"
    ```
    
    Replace:
    - `E:` with your mounted ISO drive letter
    - `V:` with your VHDX drive letter
    - Use "/Get-ImageInfo" flag first to view available editions if needed

3. **Configure boot files**:
    ```cmd
    bcdboot V:\Windows /s V: /f UEFI
    ```
    Replace `V:` with your VHDX drive letter

## Step 3: Configure Boot Entry ##

1. **Create boot entry**:
    ```cmd
    bcdedit /store V:\EFI\Microsoft\Boot\BCD /copy {default} /d "Windows 11 VHDX"
    ```
    Note the returned GUID (e.g., `{01234567-89ab-cdef-0123-456789abcdef}`)

2. **Configure the boot entry**:
    ```cmd
    bcdedit /store V:\EFI\Microsoft\Boot\BCD /set {GUID} device vhd=[D:]\VirtualDisks\Win11.vhdx
    bcdedit /store V:\EFI\Microsoft\Boot\BCD /set {GUID} osdevice vhd=[D:]\VirtualDisks\Win11.vhdx
    bcdedit /store V:\EFI\Microsoft\Boot\BCD /set {GUID} detecthal on
    ```
    
    Replace:
    - `{GUID}` with the GUID from the previous step
    - `[D:]` with the actual drive letter where your VHDX is stored

3. **Add Windows boot manager to your system's EFI partition**:
    ```cmd
    mountvol S: /s
    bcdedit /store S:\EFI\Microsoft\Boot\BCD /create /d "Windows 11 VHDX Boot" /application bootsector
    bcdedit /store S:\EFI\Microsoft\Boot\BCD /set {bootmgr} default {GUID}
    bcdedit /store S:\EFI\Microsoft\Boot\BCD /set {bootmgr} path \EFI\Microsoft\Boot\bootmgfw.efi
    ```

## Step 4: First Boot and Configuration ##

1. **Detach VHDX and restart**:
    ```cmd
    diskpart
    select vdisk file="D:\VirtualDisks\Win11.vhdx"
    detach vdisk
    exit
    
    wpeutil reboot
    ```

2. **Boot to Windows 11**:
    - Select "Windows 11 VHDX" from the boot menu
    - Complete the Windows 11 OOBE setup


