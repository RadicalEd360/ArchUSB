# ArchUSB
A fully installed, editable, USB device that runs from RAM.

This repository is a Howto Guide, Proof of Concept, and Disk Image Archive.    

Feel free to suggest any features/improvements!   

## My Inspiration for this Project
- I needed a few headless computers to manage some of my harddrives.
- I wanted to make use of all available sata ports for storage drives.
- Puppy Linux was just not cutting it for my headless network devices.

---

## Image Notes
- The default logins **root:toor** **mars:atlantis**
- Image can be written by piping gunzip into dd.
- Partition table is built for Legacy Booting
- Filesystem is ext4 with journaling disabled
- Kernel is linux-lts with headers and firmware  

You can install/update files on the USB while running from Ram with a mounted arch-chroot   
*I have written a bash script to do just this* **/usr/local/sbin/mntfs.sh**

---

# How to Create this project from Scratch
- [step 1](https://github.com/RadicalEd360/ArchUSB/blob/main/README.md#step-1) Install Arch linux to a [Removable Medium](https://wiki.archlinux.org/title/Install_Arch_Linux_on_a_removable_medium)
- [step 2](https://github.com/RadicalEd360/ArchUSB/blob/main/README.md#step-2) Install and configure Arcmags [Ramroot](https://github.com/arcmags/ramroot/blob/master/ramroot)
- [step 3](https://github.com/RadicalEd360/ArchUSB/blob/main/README.md#step-3) Edit [mkinitcpio hooks](https://wiki.archlinux.org/title/mkinitcpio#Common_hooks) to activate USB block devices
- [step 4](https://github.com/RadicalEd360/ArchUSB/blob/main/README.md#step-4) Configure the [bootloader](https://wiki.archlinux.org/title/Arch_boot_process#Boot_loader) and [fstab](https://wiki.archlinux.org/title/Fstab) to find and load from USB
- [step 5](https://github.com/RadicalEd360/ArchUSB/blob/main/README.md#step-5) Install your desired [services and features](https://wiki.archlinux.org/title/general_recommendations) to Arch linux

---
