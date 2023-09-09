# proxmoxdox
proxmox related documents to memorize some tweaks

 - [Power Consumtion](PowerConsumtion.md)
 - [Suspend guests on host shutdown](SuspendAllOnReboot.md)
 - [Migrate to larger drive v2](MigrateToLargerDrive_v2.md)
 - [Enlarge LVM VM storage disk](EnlargeVMdisk.md)
---

## Migrating Proxmox to a Larger Physical Drive

If you're interested in migrating Proxmox to a larger drive, jump directly to the section [Expansion of VM Disk Storage](#expansion-of-vm-disk-storage). But first, let's delve into the history and context of the migration process.

### Background and Context

When I began using Proxmox, it was part of my journey to find the best-fit host OS and VM software for my home server. This server was intended to host various services like web hosting, Git, and more.

I chose the GTR7 mini-PC (Ryzen 7840HS version) from Bee-Link as the hardware base. I equipped it with 64GB of DDR5-5600 RAM and later installed two 1TB NVMEs for the OS and 4TB NVMEs for data. Due to the NVME heat sink's design for one-side NVME drives, I had to acquire different silicon thermal pads to install them correctly.

During the waiting period for thermal pads, I started configuring the host OS.

### Initial OS Installation Attempts

Before installing the host OS, I used USB-connected storage to avoid interfering with internal drives lacking proper cooling.

1. **First Attempt - Ubuntu 22.04:** At that time, the CPU was very new, and Kernel 6.2.x wasn't released yet. There was no video card support, and the AMD drivers caused system crashes. Additionally, I couldn't choose the right resolution, and the system exhibited instability. Ubuntu was unsuitable.

2. **Second Attempt - Windows Server 2022:** Faced the same driver issues, had to dismiss it.

3. **First Success - Windows 10 x64:** This installation succeeded with drivers provided by Bee-Link. However, the system experienced random reboots while idle. Luckily, Bee-Link released a BIOS update that resolved the random reboot problem. The system became stable and energy-efficient (around 12 watts idle), hosting a VMWare Windows Server 2022 guest along with various services.

4. **Second Success - Ubuntu 22.04 with Kernel Update:** With kernel updates, both Linux and Ubuntu provided a stable experience. Power efficiency was close to Windows 10 x64, though slightly higher at 12-15 watts idle.

### Trade-offs and Decision-making

At this point, a trade-off emerged:

- **Windows 10 x64:** It was stable and allowed GUI tasks on the physical machine. The system's computing power supported activities like movie playback and 3D gaming. While these were occasional tasks for me, the option was there.

- **Ubuntu:** Although limiting GUI experience and overall system resource efficiency compared to Windows (it had advantages such as better hardware resource utilization). Ubuntu produced a bit more heat but offered Linux benefits.

### Discovery of Proxmox VE8

5. **Third Success - Proxmox VE8:** While exploring virtualization alternatives, I discovered Proxmox VE8 based on Linux Kernel 6.2.x. I configured it without local X-server and was amazed by the fast boot times for both the host and Windows Server 2022 guest from a single 64GB TF-card in a USB3.0 card reader. Each system boot took not longer than 10 seconds.

### Optimizing Power Consumption

However, Proxmox's power consumption was higher than desired. I implemented tweaks to reduce it:

- Adjusted CPU governors using the following command:
  ```shell
  echo "schedutil" | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
  ```
  
- Tweaked the `screenblank` parameter in Grub to set display timeout to one minute, saving additional power.

### Expansion of VM Disk Storage

As my VM storage requirements grew, I decided to move from a 64GB TF-card to a 128GB TF-card (as a test before migrating to an internal SSD). Here's how I accomplished this:

1. Booted into another Linux environment or a Linux rescue CD (I used an external USB3.0 SSD case with an Ubuntu OS).

2. Copied content from the 64GB TF-card to the 128GB TF-card using the command:
   ```shell
   sudo dd if=/dev/sde of=/dev/sdc bs=64M status=progress
   ```
   '/dev/sde' is my source drive detected
   and
   '/dev/sdc' is my destination drive detected
   
4. Fixed the GPT backup table using the command:
   ```shell
   sudo gdisk /dev/sdc
   ```
   Inside gdisk, I executed the following commands:
   ```
   p
   v
   w
   q
   ```

5. Resized the last LVM partition (in this case, `/dev/sdc3`) using the following command:
   ```shell
   sudo parted --resize -l 125G /dev/sdc3
   ```
   //TODO: make sure that syntax for command 'parted' is correct.
   
6. Shutdown the system and remove the source drive.

7. Boot into the new destination drive and access the Proxmox shell through the web interface console or SSH terminal.

8. Resized the logical partition to the physical partition size using the commands:
   ```shell
   lvresize -l +100%FREE /dev/sdc3
   lvresize -l +100%FREE /dev/mapper/pve-data
   ```

And that concludes the process of migrating Proxmox to a larger physical drive.

---

Please note that some of the commands provided in the text may have specific requirements or variations based on your system setup. Always ensure you have backups and understand the implications of each command before executing them.
