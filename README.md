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

1. Close the installer using right click in the bottom bar
2. Open terminal
3. List devices and partitions to determine which disk to use
   ```shell
   sudo fdisk -l
   ```
4. Modify partition table of the primary disk, e.g.
   ```shell
   sudo cfdisk /dev/nvme0n1
   ```
   or
   ```shell
   sudo cfdisk /dev/sda
   ```
   1. If no partition with type `EFI System` exists that is larger than `500MB`
      1. Create a new partition with `500MB`
      2. Select the partition using the up and down arrow keys
      3. Select `Type` using the left and right arrow keys and press enter
      4. Select `EFI System` and press enter
   2. Create a new partition using the remaining space
      (Required `150GB`; Recommended `250GB+`)
   3. List devices and partitions to determine the primary OS partition
      ```shell
      sudo fdisk -l
      ```
   4. Export the `DEVICE` environment variable with the OS partition, e.g.
      ```shell
      export DEVICE=/dev/nvme0n1p2
      ```
   5. To initialize the encrypted OS partition run the following commands
      1. Create encrypted partition
         ```shell
         sudo cryptsetup luksFormat "$DEVICE"
         ```
      2. Open encrypted partition
         ```shell
         sudo cryptsetup luksOpen "$DEVICE" linux-temp
         ```
      3. Create LVM
         ```shell
         sudo pvcreate /dev/mapper/linux-temp \
         && sudo vgcreate pop_os /dev/mapper/linux-temp \
         && sudo lvcreate -n root -l +100%FREE pop_os \
         && sudo mkfs -t ext4 /dev/mapper/pop_os-root
         ```
   6. Switch to installer
   7. Click through configuration until you can select between `Clean Install` and
      `Custom (Advanced)` and select the later
   8. Select the partitions
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
   9. Continue with installer steps
      1. For the username it is recommended to use the given + family name separated by an
         underscore; e.g. `fabian_haenel`

## Setup

1. Install Ansible
   ```shell
   sudo apt install python3-pip \
   && sudo pip install ansible
   ```
2. Download repository containing playbook
   ```shell
   mkdir ~/Repositories \
   && cd ~/Repositories \
   && git clone https://github.com/fast-lane-digital/setup-developer-popos.git
   ```
3. Configure PreLoader to allow secure boot
   ```shell
   ansible-playbook --ask-become-pass ~/Repositories/setup-developer-popos/playbooks/configure-secure-boot/main.yml
   ```
4. Restart into UEFI and enable secure boot
5. Enroll hashes see [Enroll UEFI Hashes](#enroll-uefi-hashes)
6. Setup system
   ```shell
   ansible-playbook --ask-become-pass ~/Repositories/setup-developer-popos/playbooks/setup/main.yml
   ```

## Enroll UEFI Hashes

### After an kernel update

You will see a screen that complains about failing to start

1. Click `OK`
2. Click on `Enroll Hash`
3. Navigate to `../`
4. Navigate to `Pop_OS-...` (... is a UUID4 and unique for each installation)
5. Navigate to `vmlinuz.efi` and press enter and confirm

### After a systemd-boot update

1. Click `OK`
2. Click on `Enroll Hash`
3. Navigate to `loader.efi` and press enter and confirm
