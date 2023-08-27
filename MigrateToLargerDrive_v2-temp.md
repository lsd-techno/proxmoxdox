## Migrating Proxmox to a Larger Physical Drive v2

>*note1:
    While making first migration to another drive I notice that drive copy speed drop down to 30+ MB/s at the end of disk copy with 'dd'.
    Guest OS boot time increased too. Overal card speed and response for 64GB card was better. That transfer rate I test in past.
    So I decide to give it another try with bigger and faster TF-Card and bigger block size and document this process more carefully this time as well.*

>*note2:
    While typing this documentary, copy process passed 50% boundary 75GB with bs=1G and copy speed drop to 55MB/s.
    That is better than previose time when speed dropped to 37MB/s after 50GB copy process.
    Total copy process took 3412 seconds for 125GB and ended with same 37MB/s speed.
    Next time need to try bigger block size to give more than 12 seconds to rest to destination card.
    In any case write speed to TF-Card should be not important when migrate to ssd.*

---

### Materials in use
1. HOST PC (Model: GTR7, CPU: 7840HS, RAM: 64GB DDR5-5600, NVME: installed 1+4TB but not involved in this experiment)
2. 2x USB 3.0 TF-Card reders
3. Source 128GB TF-Card with standalone proxmox ve8 node installed (standalone means all the resources Host OS, Guest install media ISOs and VM storage are located on this card).
4. Destination 256GB TF-Card
5. Bootable linux environment such as Linux rescue CD or Live CD or fully installed OS on removable drive (removable is optional). I use ubuntu 22.04.x (latest LTS at the moment) installed on USB attached SSD.

### Expansion of ProxMox VE8 Storage Disk partition

1. Booted into another Linux environment or a Linux rescue CD (I used an external USB3.0 SSD case with an Ubuntu OS).

2. use *'lsblk'* to identify Your drives.

3. optional use *'umount /mount/links'* if any partitions of Your *source* or *destination* drive appear to be mounted

4. Copy content from the 128GB TF-card to the 256GB TF-card using the command:
   ```shell
   sudo dd if=/dev/sde of=/dev/sdc bs=1G status=progress
   ```
   '/dev/sde' is my source drive detected
   and
   '/dev/sdc' is my destination drive detected

5. use *'lsblk'* to confirm that You can see same partiotions appear on *destination* drive.
   
6. Fix the GPT backup table using the command:
   ```shell
   sudo gdisk /dev/sdc
   ```
   Inside gdisk, I executed the following commands:
   ```
   p
   v
   w yes, yes
   ```

7. Resized the last LVM partition (in this case, `/dev/sdc3`) using the following command:
   ```shell
   sudo parted /dev/sdc3 resizepart 3
   ```
   then type: *100%*
   then click enter.

8. use *'lsblk'* to confirm Your destination partition occupy all free space on drive
   
9. Shutdown the system and remove the source drive.

### Expansion of VM Disk Storage to entire partition

1. Boot into the new destination drive and access the Proxmox shell through the web interface console or SSH terminal.

2. use *'lsblk'* to confirm which one is Your LVM partition.

   ![1693163408624](https://github.com/lsd-techno/proxmoxdox/assets/6795932/5fd5b094-7669-4140-a380-292a448250c6)

3. Resize the phisical volume to the physical partition size with following commands:
   ```shell
   pvresize /dev/sdc3
   pvdisplay
   ```
   ![1693164905851](https://github.com/lsd-techno/proxmoxdox/assets/6795932/fe39380a-3173-410e-8df0-d9a49ffacf8a)

4. Optional resize *'pve-root'* logical disk if You want to store more or less ISO files there:
   ```shell
   lvresize -L +5G /dev/mapper/pve-root
   ```
   *remember to free up space before doing *'lvresize'* if You are going to shrink partition*
   
   ![1693165162159](https://github.com/lsd-techno/proxmoxdox/assets/6795932/f0a30852-1614-44f2-825a-86d5b311fcdc)


6. Resize *'pve-data'* logical disk to rest of free spcae on logical partition using the command:
   ```shell
   lvresize -l +100%FREE /dev/mapper/pve-data
   ```
   *that is the place where all Your VM disks are located by default*

   ![1693165212987](https://github.com/lsd-techno/proxmoxdox/assets/6795932/f73c3697-25ab-4cda-a255-dac1fa8d6f3a)

7. Now in ProxMox web interface can see increased storage capacity:
   
   ![1693165417234](https://github.com/lsd-techno/proxmoxdox/assets/6795932/319834dc-623d-4991-80e7-c6136ddbd8ef)


And that concludes the process of migrating Proxmox to a larger physical drive.

---

Please note that some of the commands provided in the text may have specific requirements or variations based on your system setup. Always ensure you have backups and understand the implications of each command before executing them.
