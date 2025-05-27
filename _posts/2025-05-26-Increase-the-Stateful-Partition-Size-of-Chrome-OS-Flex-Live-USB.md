---
title: "Increase the Stateful Partition Size of Chrome OS Flex Live USB"
date: 2025-05-26 22:30:00 +0000
categories: [ChromeOS Flex, Resize]
tags: [ChromeOS, Live USB]
author: srodi
comments: true
---

When you create a Chrome OS Flex live USB drive using the official Chromebook Recovery Utility extension, you might notice a frustrating limitation: the recovery image doesn't utilize the full capacity of your USB stick. If you're using a 64GB or 128GB USB drive, Chrome OS Flex might only use a small portion of that space, leaving the rest inaccessible.

This becomes particularly problematic when you want to install Linux applications or store additional files on your Chrome OS Flex system, as the limited stateful partition quickly runs out of space.

## The Solution: Manual Partition Expansion

There's a way to expand the Chrome OS Flex image before flashing it to your USB drive. This process involves modifying the recovery image file to utilize more of your USB drive's capacity.

## Steps

### 1. Download the Chrome OS Flex Recovery Image

First, obtain the official Chrome OS Flex recovery image:
- Visit: https://chromiumdash.appspot.com/serving-builds?deviceCategory=ChromeOS%20Flex
- Download the recovery image

### 2. Prepare Your Linux Environment

Open a Linux terminal (or WSL2 if using Windows) and navigate to the directory containing your image file:

```bash
cd /path/to/your/image/directory
```

### 3. Expand the Image Size

Use the `truncate` command to increase the image file size. This adds free space that we can later allocate to partitions:

```bash
truncate -s +50G chromeos.bin
```

> **Note:** Replace `chromeos.bin` with your actual image filename. Adjust `50G` based on your needs and USB drive capacity.

### 4. Set Up Loop Device

Create a loop device to work with the image file as if it were a physical disk:

```bash
sudo losetup -Pf chromeos.bin
```

### 5. Identify the Loop Device

Find out which loop device was assigned:

```bash
lsblk
```

Look for entries like `loop0`, `loop1`, etc. Note the device name for subsequent commands.

### 6. Repair the GPT Partition Table

Check the current partition table status:

```bash
sudo cgpt show /dev/loop0
```

> **Note:** Replace `loop0` with your actual loop device name.

If you see "Invalid info" or similar errors, repair the partition table:

```bash
sudo cgpt repair /dev/loop0
```

### 7. Fix Partition Table Issues with gdisk

Use gdisk to ensure the partition table is properly structured:

```bash
sudo gdisk chromeos.bin
```

Within gdisk, execute these commands:
- Press `p` to print the partition table
- Press `x` to enter the expert menu
- Press `e` to access extra functionality
- Press `w` to write changes and exit

### 8. Expand the Chrome OS Flex Partition

Use cgpt to modify partition 1:

```bash
sudo cgpt add -i 1 -s 83886080 /dev/loop0
```

> **Important:** The value `83886080` represents approximately 40GB in sectors. Adjust this value based on your desired partition size. You can calculate sectors by: `desired_size_in_GB × 1024 × 1024 × 2`

### 9. Verify the Changes

Confirm that the partition has been expanded correctly:

```bash
sudo cgpt show /dev/loop0
```

### 10. Repair and Resize the Filesystem

Check the filesystem for any issues:

```bash
sudo e2fsck -f /dev/loop0p1
```

Resize the filesystem to use the newly allocated space:

```bash
sudo resize2fs /dev/loop0p1
```

### 11. Clean Up

Detach the loop device:

```bash
sudo losetup -d /dev/loop0
```

If you have any mounted partitions, unmount them:

```bash
sudo umount /path/to/mountpoint
```

### 12. Create the Bootable USB

Now you can create your bootable USB with the expanded image:

1. Insert your USB drive
2. Open Chrome browser
3. Launch the Chromebook Recovery Utility extension
4. Click the settings icon (gear icon in the top right)
5. Select "Use local image"
6. Browse and select your modified Chrome OS Flex image
7. Select your USB drive as the target
8. Start the recovery process
