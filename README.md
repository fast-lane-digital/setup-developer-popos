# Set up Pop!Os 22.04 LTS

This repository provides instructions, commands, scripts to set up Pop!Os 22.04 LTS for
Python and NodeJS developers.

## Preparation

1. Boot into Windows and login using your local administrator account
2. Open start menu and type `Partition` which should brink up
   `Create and format hard disk partitions`
3. Shrink the largest partition by right-clicking and selecting `Shrink Volume...`
   At least `150GB` should be made available, recommended are `250GB`
4. Exit and reboot the computer
5. Disable `Secure Boot` in the UEFI
6. Insert USB Stick with Pop!Os installer
7. Select installer as boot device

## Installation

1. Ignore installer menu for now
2. Open terminal
3. List devices and partitions `sudo fdisk -l`
4. Modify partition table of the primary disk
   e.g. `sudo cfdisk /dev/nvme0n1` or `sudo cfdisk /dev/sda`
   1. If no partition with type `EFI System` exists that is larger than 500MB
      1. Create a new partition with 500MB
      2. Select the partition using the up and down arrow keys
      3. Select `Type` using the left and right arrow keys and press enter
      4. Select `EFI System` and press enter
   2. Create a new partition using the remaining space (needs to be at least 150GB
      large)
5. List devices and partitions `sudo fdisk -l` to determine the primary OS partition
6. Run `export PARTITION=/dev/nvme0n1p2` and replace `nvme0n1p2` with the correct
   partition from the previous step
7. To initialize the encrypted OS partition run the following commands
   1. Create encrypted partition
      `sudo cryptsetup luksFormat "/dev/$PARTITION"`
   2. Open encrypted partition
      `sudo cryptsetup luksOpen "/dev/$PARTITION" linux-temp`
   3. Create LVM
      ```shell
      sudo pvcreate /dev/mapper/linux-temp
      sudo vgcreate pop_os /dev/mapper/linux-temp
      sudo lvcreate -n root -l +100%FREE pop_os
      sudo mkfs -t ext4 /dev/mapper/pop_os-root
      ```
8. Switch to installer
9. Click through configuration until you can select between `Clean Install` and
   `Custom (Advanced)` and select the later
10. Select the partitions
    1. Depending on the partitioning steps select the `EFI` partition
       1. If you had to create a new one
          ```
          Format: true
          Use as: Boot (/boot/efi)
          Filesystem: fat32
          ```
       2. If the existing partition was large enough
          ```
          Format: false <- REALLY IMPORTANT OTHERWISE WINDOWS WONT BOOT
          Use as: Boot (/boot/efi)
          Filesystem: fat32
          ```
    2. Select the OS partition (easily identifiable by the lock symbol)
11. Continue with installer steps
    1. For the username it is recommended to use the given + family name separated by an
       underscore; e.g. `fabian_haenel`

## Setup