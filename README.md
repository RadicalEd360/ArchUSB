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

## Disk Image Notes
- The Compressed Image is just under 1G, and requires almost 3G of USB space.
- To run this from ram, you should have atleast 4G total RAM.
- Github requires a project called **git-lfs** to download/upload files larger than 100M. (basically stores them somewhere else.)
- The default logins are **root:toor** **mars:atlantis**
- Image can be directly written by piping gunzip into dd `gunzip -k file.img.gz | sudo dd of=/dev/sXX bs=512M status=progress`.
- Partition table is built for Legacy Booting
- Filesystem is ext4 with journaling disabled
- Kernel is linux-lts with headers and firmware  
- To allocate more RAM, edit /etc/ramroot.conf, then issue ramroot -E

You can install/update packages/files onto the USB while running from Ram with a mounted arch-chroot   
*I have written a bash script to do just this* **/usr/local/sbin/mntfs.sh**

---

# How to Create this project from Scratch
- [Step 1](https://github.com/RadicalEd360/ArchUSB#1-install-arch-to-a-removable-medium) Install Arch linux to a [Removable Medium](https://wiki.archlinux.org/title/Install_Arch_Linux_on_a_removable_medium)
- [Step 2](https://github.com/RadicalEd360/ArchUSB#2-install-ramroot) Install and configure Arcmags [Ramroot](https://github.com/arcmags/ramroot/blob/master/ramroot)
- [Step 3](https://github.com/RadicalEd360/ArchUSB#3-configure-mkinitcpioconf) Edit [mkinitcpio hooks](https://wiki.archlinux.org/title/mkinitcpio#Common_hooks) to activate USB block devices
- [Step 4](https://github.com/RadicalEd360/ArchUSB#4-configure-a-bootloader) Configure the [bootloader](https://wiki.archlinux.org/title/Arch_boot_process#Boot_loader) and [fstab](https://wiki.archlinux.org/title/Fstab) to find and load the USB
- [Step 5](https://github.com/RadicalEd360/ArchUSB#5-finishing-up) Install your desired [services and features](https://wiki.archlinux.org/title/general_recommendations) to Arch linux

---

### 1. Install Arch to a Removable Medium
There are many ways to install Arch Linux.  
An easy way to install directly to USB is with Virtualbox.
- create a raw vmdk that points to the USB Drive.  
- Create a VM and use the vmdk as your harddrive.
- insert Install disk, boot, and continue like normal.  

This step involves a basic install up until you reach the chroot part.  
You will want to use ext2 partitions rather than ext4 to avoid taxing the USBs read/writes via fsck. This greatly improves the devices longterm health.  
You will also want to LABEL your partitions for boot versatility.  
Continue as Normal until you are done installing the kernel, partitioning, setting your timezone info etc.

Before exiting the chroot, continue to step 2.

---

### 2. Install Ramroot
Ramroot exists in the AUR  
you do not need to install an AUR helper, although you can if you want.  

Just install git and use makepkg instead of an AUR helper.  
makepkg requires a lesser priviledged user to start building, and then later uses sudo to install the finished package.  
We can use the git user to do this without needing to create a new user by giving it sudo access with visudo.  

```
visudo            # give nopasswd access to git user - git ALL=(ALL:ALL) NOPASSWD: ALL
sudo -u git bash  # open a shell as the git user

git clone $aururl # download any AUR packages
cd $aurpkg        # We specifically are installing ramroot
makepkg -si       # repeat these steps
cd ..             # for each AUR package

exit              # return to root chroot for further configuration
visudo            # undo sudoer access for git user.
```

After installing ramroot, we can configure it to our liking by editing **/etc/ramroot.conf**  
the default configuration is fine, however you may want to set the ps_timeout and ps_default variables.  
I have set the ps_timeout to 1 and the ps_default to yes in order to ensure my USB copies into ram quickly (if able) without any user intervention.

After any changing any configuration, you must always activate your new settings with `ramroot -E`

---

### 3. Configure mkinitcpio.conf
at this moment if we rebooted, the ramdisk would never find our USB partitions  
this is because the system is configured to power on **block** devices after trying to find the root partition through sata.  

to fix this problem we must edit /etc/mkinitcpio.conf and rearrange the order of events known as *hooks*  
move the **block hook** closer to the beginning just after base. the following setup works for me.  
`HOOKS=(base block udev ramroot autodetect modconf filesystems keyboard fsck)`

after doing this we will need to regenerate the initramfs by issuing either `ramroot -E` or `mkinitcpio -P`  
the system will now check connected usb devices for the root partition at boot.

---

### 4. configure a bootloader
We have chosen grub as our bootloader.  
your chosen bootloader may be different, please reffer to their documentation.  
basically the goal here is to make sure they find the right device to boot from.  

this is the point where labeling your partition has the advantage over UUIDs  
USB devices give out faster than SSDs and as such we will likely install multiple USBs  
a dedicated UUID will eventually require a reconfiguration so we do it now instead.  

we first edit /etc/fstab and point / to our label like so:  
`LABEL=ArchUSB          /               ext4            rw,relatime     0 1`  
ramroot comments out this line and sets / to the zram while booting, it is good to have this in case we don't want to run from RAM.  

then we edit /etc/default/grub and change settings to our liking such as the timeout.  
```
GRUB_DEFAULT=0
GRUB_TIMEOUT=1
export GRUB_DISABLE_UUID=true
export GRUB_DEVICE="LABEL=ArchUSB"
```
the last two lines are the most important and are self explanatory.  
we tell grub to look for a specific LABEL instead of searching for UUIDs.  

we have one last step of configuring before we can properly boot with grub on a USB.  
we must first **generate the grub.cfg then edit that file** as I could not find a way to do this properly within /etc/default/grub  
I know its not ideal to edit grub.cfg as its not very dynamic, However I repeat, I did not find a way to do this within /etc/default/grub  
If you know how to override grubs heuristics algorithm and set a custom root, please share. Thank you.  

**Generate the grub.cfg** by issuing `grub-mkconfig -o /boot/grub/grub.cfg`  
then edit the file and **change all lines that look like** `set root='hdX,msdos1'` to `set root='hd0,msdos1'`  
when booting from a USB, the device is **always located at hd0**, so we must tell it to look there or it will not boot.

---

### 5. Finishing Up
**Congratulations, you may continue the normal Arch install process** by installing your desired features!  
I installed some utilities i like such as tmux, ranger, wifi firmware, networkmanager, openssh and so on.  
I then configured networkmanager to auto connect to wifi and openssh to autostart as a service.  
I was happy with it being headless and did not install a desktop environment, your purposes may be different than mine.  
**You may want to install a desktop environment**.  

Keep in mind that any package you install **adds space to the disk and will take more time to copy into ram**.    

After you are done installing packages,  
**I highly reccomend you clear pacmans cache** with `pacman -Scc`.  
this will free up a large amount of data.

---

Please feel free to suggest any Features/Improvements.    
Thank You -Edward


![alt text](https://images.pexels.com/photos/220357/pexels-photo-220357.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1 "Image Credit: Hitarth Jadhav on pexels.com")
