## Manage power consumption

---

By default, Proxmox runs the host machine's CPU at maximum power, and there are no options to modify that behavior.
I want my hardware to run in balanced mode, having enough CPU power when required and saving as much power as possible during idle moments.

Here is a way to implement the modern built-in 'balanced mode' into the Linux kernel:

1. install 'cpufrequtils'
   ```shell
   apt install cpufrequtils
   ```
2. create configuration file
   ```shell
   cat << 'EOF' > /etc/default/cpufrequtils
   GOVERNOR="schedutil"
   EOF
   ```
3. confirm configuration file content
   ```shell
   cat /etc/default/cpufrequtils
   ```
![1694242147645](https://github.com/lsd-techno/proxmoxdox/assets/6795932/4c35e157-31bc-4f85-b408-1bec9538f4a8)

4. after reboot, check for the current power policy.
   ```shell
   cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
   ```
![1694242292473](https://github.com/lsd-techno/proxmoxdox/assets/6795932/4e4589ea-5fa7-4fdb-b5a2-01898269ae27)

5. modify grub config, add 'consoleblank=60' to 'GRUB_CMDLINE_LINUX_DEFAULT' so display go to power off mode after 60 seconds
   ```shell
   nano /etc/default/grub
   ```
   
![1694245795719](https://github.com/lsd-techno/proxmoxdox/assets/6795932/2aa0402f-1984-477b-b10c-2ad6dc9206ac)


6. run following command after grub config update:
   ```shell
   update-grub
   ```

7. after reboot, confirm that display go to power save mode after 60 seconds.
   
