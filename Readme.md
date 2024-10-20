# Actionpak System Information Script

## Overview

Tired of manually checking IP addresses and port numbers for your LXC containers? This script automates that by listing useful infomatoon whenever you log into your LXC.
 streamlines system management by providing a quick overview of system status, security insights, performance metrics, network information, and maintenance requirements.
 By automating the collection and display of crucial system data, ActionPak saves time and effort. Additionally, its customizable nature, cross-distribution compatibility,
 and educational value make it a versatile and user-friendly tool for both experienced users and those new to Linux.

![Pasted Graphic 4](https://github.com/user-attachments/assets/7860c376-545e-4f9f-be75-2e66b65b4ef0)

## Features

- Displays a custom ASCII art banner
- Shows basic system information (hostname, IP address, system load, memory usage, etc.)
- Lists open ports and categorizes them by service type
- Provides web access URLs for open ports (HTTP and HTTPS)
- Reports on system updates, failed login attempts, and active connections
- Lists network interfaces
- Automatically runs on user login
- Easy installation and uninstallation process

## Installation

1. Save the script to a file, e.g., `install_actionpak_system_info.sh`
2. Make it executable:
   ```
   chmod +x install_actionpak_system_info.sh
   ```
3. Run it with sudo:
   ```
   sudo ./install_actionpak_system_info.sh
   ```

## Usage

After installation, Actionpak will run automatically every time a user logs in. You can also run it manually at any time:

```
sudo advanced_system_info.sh
```

## Uninstallation

To remove Actionpak from your system, run:

```
sudo advanced_system_info.sh --uninstall
```

## Installer 

```bash
#!/bin/bash

# Check if the script is being run as root
if [ "$EUID" -ne 0 ]; then
    echo "Please run as root or with sudo"
    exit 1
fi

# Uninstall function
uninstall() {
    rm -f /usr/local/bin/advanced_system_info.sh
    rm -f /etc/profile.d/run_advanced_system_info.sh
    echo "Actionpak system information script has been uninstalled."
    exit 0
}

# Check for uninstall argument
if [ "$1" = "--uninstall" ]; then
    uninstall
fi

# Save this script to /usr/local/bin/advanced_system_info.sh
cat > /usr/local/bin/advanced_system_info.sh << 'EOL'
#!/bin/bash

# Function to get open ports, excluding 22 and 25
get_open_ports() {
    netstat -tuln | grep LISTEN | awk '{print $4}' | awk -F: '{print $NF}' | sort -n | uniq | grep -vE '^22$|^25$' | tr '\n' ' '
}

# Function to get web access URLs
get_web_access() {
    local ip=$1
    local ports=$2
    local protocol=$3
    for port in $ports; do
        echo "$protocol://$ip:$port"
    done
}

# Function to categorize ports
categorize_ports() {
    local ports="$1"
    local web_ports=$(echo "$ports" | grep -E "80 |443 |8080 |8443 ")
    local db_ports=$(echo "$ports" | grep -E "3306 |5432 |27017 ")
    local mail_ports=$(echo "$ports" | grep -E "587 |993 |995 ")
    local ftp_ports=$(echo "$ports" | grep -E "21 ")
    local rdp_ports=$(echo "$ports" | grep -E "3389 |5900 ")
    
    [ ! -z "$web_ports" ] && echo "Web servers: $web_ports"
    [ ! -z "$db_ports" ] && echo "Databases: $db_ports"
    [ ! -z "$mail_ports" ] && echo "Mail: $mail_ports"
    [ ! -z "$ftp_ports" ] && echo "FTP: $ftp_ports"
    [ ! -z "$rdp_ports" ] && echo "Remote desktop: $rdp_ports"
}

# Get system information
HOSTNAME=$(hostname)
IP_ADDRESS=$(hostname -I | awk '{print $1}')
OPEN_PORTS=$(get_open_ports)
LOAD=$(uptime | awk -F'load average:' '{print $2}' | sed 's/^ *//')
MEM_USAGE=$(free | awk '/Mem:/ {printf("%.1f%%", $3/$2 * 100)}')
DISK_USAGE=$(df -h / | awk '/\// {print $5}')
PROCESSES=$(ps aux | wc -l)
UPDATES=$([ -x "$(command -v apt)" ] && apt list --upgradable 2>/dev/null | grep -c upgradable || echo "N/A")
FAILED_LOGINS=$(journalctl _SYSTEMD_UNIT=sshd.service | grep "Failed password" | wc -l)
ACTIVE_CONNECTIONS=$(netstat -ant | grep ESTABLISHED | wc -l)
FIREWALL_RULES=$(iptables -L | grep -c '^ACCEPT\|^REJECT\|^DROP')

# Get web access URLs
HTTPS_ACCESS=$(get_web_access $IP_ADDRESS "$OPEN_PORTS" "https")
HTTP_ACCESS=$(get_web_access $IP_ADDRESS "$OPEN_PORTS" "http")

# Display Actionpak banner and advanced system information
cat << EOF
$(tput bold)$(tput setaf 2)
 █████╗  ██████╗████████╗██╗ ██████╗ ███╗   ██╗██████╗  █████╗ ██╗  ██╗
██╔══██╗██╔════╝╚══██╔══╝██║██╔═══██╗████╗  ██║██╔══██╗██╔══██╗██║ ██╔╝
███████║██║        ██║   ██║██║   ██║██╔██╗ ██║██████╔╝███████║█████╔╝ 
██╔══██║██║        ██║   ██║██║   ██║██║╚██╗██║██╔═══╝ ██╔══██║██╔═██╗ 
██║  ██║╚██████╗   ██║   ██║╚██████╔╝██║ ╚████║██║     ██║  ██║██║  ██╗
╚═╝  ╚═╝ ╚═════╝   ╚═╝   ╚═╝ ╚═════╝ ╚═╝  ╚═══╝╚═╝     ╚═╝  ╚═╝╚═╝  ╚═╝
                                                                by Bas
$(tput sgr0)

Welcome to $HOSTNAME, $(lsb_release -d | cut -f2)
System information as of $(date)

System load:  $LOAD
Memory usage: $MEM_USAGE
Processes:    $PROCESSES
Disk usage:   $DISK_USAGE
IP address:   $IP_ADDRESS

Open ports and services:
$(echo "$OPEN_PORTS" | tr ' ' '\n' | sed 's/^/  /')

Categorized ports:
$(categorize_ports "$OPEN_PORTS" | sed 's/^/  /')

Web access (HTTPS):
$(echo "$HTTPS_ACCESS" | sed 's/^/  /')

Web access (HTTP):
$(echo "$HTTP_ACCESS" | sed 's/^/  /')

Additional Information:
- Available system updates: $UPDATES
- Failed login attempts: $FAILED_LOGINS
- Active network connections: $ACTIVE_CONNECTIONS
- Firewall rules count: $FIREWALL_RULES

Network Interfaces:
$(ip -o addr show | awk '{print "  " $2 ": " $4}' | cut -d/ -f1)

For more detailed system information, try: htop, vmstat, iostat, netstat, ss
EOF
EOL

# Make the script executable
chmod +x /usr/local/bin/advanced_system_info.sh

# Create a script in /etc/profile.d/ to run the main script on login
cat > /etc/profile.d/run_advanced_system_info.sh << 'EOL'
#!/bin/bash
if [ -x /usr/local/bin/advanced_system_info.sh ]; then
    /usr/local/bin/advanced_system_info.sh
fi
EOL

# Make the profile.d script executable
chmod +x /etc/profile.d/run_advanced_system_info.sh

echo "Actionpak system information script has been installed and set up to run on user login."
echo "The main script is located at: /usr/local/bin/advanced_system_info.sh"
echo "A launcher script has been added to: /etc/profile.d/run_advanced_system_info.sh"
echo "You can run it manually at any time with: sudo advanced_system_info.sh"
echo "To uninstall, run: sudo advanced_system_info.sh --uninstall"

# Run the script for immediate output
/usr/local/bin/advanced_system_info.sh
```

## Requirements

- Linux-based operating system only tested LXC
- Root or sudo access for installation and execution
- Basic system utilities (bash, netstat, iptables, etc.)


## Author

Developed by Bas

---

