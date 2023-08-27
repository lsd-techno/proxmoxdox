## Migrating Proxmox to a Larger Physical Drive v2

>*note1:
    While making first migration to another drive I notice that drive copy speed drop down to 30+ MB/s at the end of disk copy with 'dd'.
    Guest OS boot time increased too. Overal card speed and response for 64GB card was better. That transfer rate I test in past.
    So I decide to give it another try with bigger and faster TF-Card and bigger block size and document this process more carefully this time as well.*

>*note2:
        while typing this documentary, copy process passed 50% boundary 75GB with bs=1G and copy speed drop to 55MB/s.
        that is better than previose time when speed dropped to 37MB/s after 50GB copy process.
        total copy process took 3412 seconds for 125GB and ended with same 37MB/s speed.
        next time need to try bigger block size.*

---

### Materials in use
1. HOST PC (Model: GTR7, CPU: 7840HS, RAM: 64GB DDR5-5600, NVME: installed 1+4TB but not involved in this experiment)
2. 2x USB 3.0 TF-Card reders
3. Source 128GB TF-Card with standalone proxmox ve8 node installed (standalone means all the resources Host OS, Guest install media ISOs and VM storage are located on this card).
4. Destination 256GB TF-Card
5. Bootable linux environment such as Linux rescue CD or Live CD or fully installed OS on removable drive (removable is optional). I use ubuntu 22.04.x (latest LTS) installed on USB attached SSD.
6. 

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
