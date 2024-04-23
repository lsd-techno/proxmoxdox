---

### Adding Automatic Processing of Unauthorized Access

Assume we have created a `blist` list in `Datacenter->Firewall->IPSet` for future use, to filter access to your environment.

To parse the firewall log and add all suspect records—for example, those that are not allowed on the current `node` level (e.g., those that appear in the log with `policy DROP: IN`)—we can create and use the following script:

## Parse on Demand

**Create the file:**

```bash
nano /usr/local/bin/blist-scan-last.sh
```

**Enter the following script into the file:**

```bash
#!/bin/bash

LOG_FILE="/var/log/pve-firewall.log"
BLOCKLIST="blist"

# Function to check if IP is already in the blocklist
ip_exists_in_blocklist() {
    ip=$1
    pvesh get /cluster/firewall/ipset/${BLOCKLIST} | grep -wq "${ip}"
    return $?
}

# Function to add IP to blocklist with a comment
add_to_blocklist() {
    ip=$1
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    comment="IP: ${ip} port-scan ${timestamp}"
    if ip_exists_in_blocklist "$ip"; then
        echo "IP ${ip} already exists in ${BLOCKLIST}."
    else
        pvesh create /cluster/firewall/ipset/${BLOCKLIST} --cidr ${ip} --comment "${comment}"
        echo "Blocked and added IP: ${ip} with comment '${comment}' to ${BLOCKLIST}."
    fi
}

# Process all past entries in the firewall log for 'policy DROP: IN'
grep "policy DROP: IN" $LOG_FILE | grep -oP '(?<=SRC=)\d+\.\d+\.\d+\.\d+' | sort -u | while read -r ip ; do
    add_to_blocklist "$ip"
done
```

**Make it executable:**

```bash
chmod +x /usr/local/bin/blist-scan-last.sh
```

**Execute it:**

```bash
/usr/local/bin/blist-scan-last.sh
```

## Monitor for New Entries Whenever They Appear

The process and script are similar but involve extra steps.

**Create the file:**

```bash
nano /usr/local/bin/blist-scan-monitor.sh
```

**Enter the following script into the file:**

```bash
#!/bin/bash

LOG_FILE="/var/log/pve-firewall.log"
BLOCKLIST="blist"

# Function to check if IP is already in the blocklist
ip_exists_in_blocklist() {
    ip=$1
    pvesh get /cluster/firewall/ipset/${BLOCKLIST} | grep -wq "${ip}"
    return $?
}

# Function to add IP to blocklist with a comment
add_to_blocklist() {
    ip=$1
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    comment="IP: ${ip} port-scan ${timestamp}"
    if ip_exists_in_blocklist "$ip"; then
        echo "IP ${ip} already exists in ${BLOCKLIST}."
    else
        pvesh create /cluster/firewall/ipset/${BLOCKLIST} --cidr ${ip} --comment "${comment}"
        echo "Blocked and added IP: ${ip} with comment '${comment}' to ${BLOCKLIST}."
    fi
}

# Monitor log file for new 'policy DROP: IN' entries and extract IPs
tail -Fn0 $LOG_FILE | while read line ; do
    echo "$line" | grep "policy DROP: IN" | grep -oP '(?<=SRC=)\d+\.\d+\.\d+\.\d+' | while read -r ip ; do
        add_to_blocklist "$ip"
    done
done
```

**Make it executable:**

```bash
chmod +x /usr/local/bin/blist-scan-monitor.sh
```

**Add a service entry:**

```bash
nano /etc/systemd/system/blist-scan.service
```

**Enter the following into the service file:**

```bash
[Unit]
Description=Custom Script for Automatic Blacklisting After Proxmox VE Services
After=pveproxy.service

[Service]
Type=simple
ExecStart=/usr/local/bin/blist-scan-monitor.sh

[Install]
WantedBy=multi-user.target
```

**Activate and start the service:**

```bash
systemctl daemon-reload
systemctl enable blist-scan.service
systemctl start blist-scan.service
```

**Check the service status:**

```bash
systemctl status blist-scan.service
```
