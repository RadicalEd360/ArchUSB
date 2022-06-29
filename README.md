# ArchUSB
Arch, fully installed onto a USB drive that runs completely from RAM.

This repository is:  
- A Howto Guide
- A Proof of Concept 

Credits to [Arcmags](https://github.com/arcmags) for writing [Ramroot](https://github.com/arcmags/ramroot)  
as well as the [Arch Community](https://bbs.archlinux.org/) and [Arch Wiki](https://wiki.archlinux.org/)

Feel free to suggest any features/improvements!   

## My Inspiration for this Project
- I needed a few headless computers to manage some of my harddrives.
- I wanted to make use of all available sata ports for storage drives.
- Puppy Linux was just not cutting it for my headless network devices.

---

## Image Notes
- Github requires a project called git-lfs for files larger than 100M
- The default logins are **root:toor** **mars:atlantis**
- Image can be directly written by piping gunzip into dd.
- Partition table is built for Legacy Booting
- Filesystem is ext4 with journaling disabled
- Kernel is linux-lts with headers and firmware  
- To allocate more RAM, edit /etc/ramroot.conf, then issue ramroot -E

You can install/update packages/files onto the USB while running from Ram with a mounted arch-chroot   
*I have written a bash script to do just this* **/usr/local/sbin/mntfs.sh**

---

# How to Create this project from Scratch
- [Step 1](https://github.com/RadicalEd360/ArchUSB/blob/main/README.md#step-1) Install Arch linux to a [Removable Medium](https://wiki.archlinux.org/title/Install_Arch_Linux_on_a_removable_medium)
- [Step 2](https://github.com/RadicalEd360/ArchUSB/blob/main/README.md#step-2) Install and configure Arcmags [Ramroot](https://github.com/arcmags/ramroot/blob/master/ramroot)
- [Step 3](https://github.com/RadicalEd360/ArchUSB/blob/main/README.md#step-3) Edit [mkinitcpio hooks](https://wiki.archlinux.org/title/mkinitcpio#Common_hooks) to activate USB block devices
- [Step 4](https://github.com/RadicalEd360/ArchUSB/blob/main/README.md#step-4) Configure the [bootloader](https://wiki.archlinux.org/title/Arch_boot_process#Boot_loader) and [fstab](https://wiki.archlinux.org/title/Fstab) to find and load from USB
- [Step 5](https://github.com/RadicalEd360/ArchUSB/blob/main/README.md#step-5) Install your desired [services and features](https://wiki.archlinux.org/title/general_recommendations) to Arch linux

---

###1-Install Arch to a Removable Medium
There are many ways to install Arch Linux.  
An easy way to install directly to USB is with Virtualbox.
- create a raw vmdk that points to the USB Drive.  
- Create a VM and use the vmdk as your harddrive.
- insert Install disk, boot, and continue like normal.  

This step involves a basic install up until you reach the chroot part.
You will want to LABEL your partitions for versatility.
Continue Install as Normal until you are done installing the kernel and setting your timezone info.

Before exiting the chroot, continue to step 2.

###2-Install Ramroot
Ramroot exists in the AUR  
you do not need to install an AUR helper, you can if you want  

just install git and use makepkg instead  
makepkg requires a lesser priviledged user to run and internally uses sudo pacman to install 
installing git just happens to create the git user that we can use to do this.

```
visudo            # give sudoer access to git user
sudo -u git bash  # open shell as git user
makepkg -si       # repeat for each of your AUR packages
exit              # return to chroot
visudo            # undo sudoer access
```

