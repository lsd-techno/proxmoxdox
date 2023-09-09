## Automatic guest suspend on host system reboot or shutdown

---

By default, Proxmox does not have any settings to configure guest 'suspend' on host shutdown.
If you have more than one guest VM running on your server and want to reboot it for hardware maintenance, it would be annoying to suspend all guest VMs manually.
It would be a great option if each VM could have such an option in the settings.

Here is a temporary workaround to suspend all running VMs on host shutdown:

1. create 'suspend-all' script
   ```shell
   nano /var/lib/vz/snippets/pve-suspend-all
   ```
   insert script body
   ```shell
   #!/bin/bash

   qm list | grep running | awk -F'[^0-9]*' '$0=$2' | while read -r vm_id; do qm suspend $vm_id --todisk 1; done;
   ```
   chmod it 
   ```shell
   chmod 755 /var/lib/vz/snippets/pve-suspend-all
   ```

2. modify 'pve-manager.services' to run suspend-all script on host shutdown
   ```shell
   nano /etc/systemd/system/pve-manager.service
   ```
   add string to run Your script in first line of 'ExecStop' section
   ```shell
   ExecStop=/var/lib/vz/snippets/pve-suspend-all
   ```
   ![1694243419679](https://github.com/lsd-techno/proxmoxdox/assets/6795932/28737db3-3e42-410a-98b3-5a3b49876648)
