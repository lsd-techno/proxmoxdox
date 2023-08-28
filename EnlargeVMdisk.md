## Enlarge gust VM disk

---

### Enlarge/Extend gust VM virtual phisical disk storage on LVM storage

1. Optional Shut down guest. In most cases will require reboot guest to let it recognize disk size change, so better to do that now.

2. Locate virtual drive that need to expand with *'lvdisplay'* command. For example: /dev/pve/vm-100-disk-1

3. Resize logical volume to 20GB:
   ```shell
   lvresize -L 20G /dev/pve/vm-100-disk-1
   ```

4. Fix disk integrity inside logical volume:
   ```shell
   gdisk /dev/pve/vm-100-disk-1
   ```
   ```shell
   p
   v
   w yes, yes
   ```

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

7. Notify kernel about partition change:
   ```shell
   partx -u /dev/pve/vm-100-disk-1
   ```
