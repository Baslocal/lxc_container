# LXC Image Builder for Proxmox

This script automates the process of creating and renaming LXC container backups in Proxmox, effectively turning them into reusable templates.
The LXC Image Builder script is a powerful tool that automates the process of creating custom LXC (Linux Container) templates in Proxmox VE. 
By automating the creation and configuration of these templates, it significantly streamlines the deployment of new LXC containers, ensuring consistency,
efficiency, and security.

## Overview

The LXC Image Builder script performs the following actions:
1. Prompts the user for a container number
2. Creates a backup of the specified container
3. Moves to the backup directory
4. Finds the most recent backup file for the given container
5. Prompts the user for a new template name
6. Renames the backup file to the new template name

## Proxmox Environment

The script is designed to work in a Proxmox Virtual Environment. Here's a snapshot of the Proxmox web interface showing various containers:

![image](https://github.com/user-attachments/assets/42fb22f2-0c6f-49eb-8116-4e39746a303f)

<img width="1048" alt="Pasted Graphic 1" src="https://github.com/user-attachments/assets/e54ced2d-f76d-47a3-bab1-3aee3ecfd3cd">

<img width="1043" alt="Pasted Graphic 2" src="https://github.com/user-attachments/assets/9bb13fd3-1a2a-4376-b558-54977bc73978">

## Script

Here's the bash script that performs the LXC image building process:

```bash
#!/bin/bash

# Prompt for container number
read -p "Enter container number: " ctnumber

# Perform backup
vzdump $ctnumber --mode stop --compress gzip --dumpdir /var/lib/vz/template/cache

# Change to the backup directory
cd /var/lib/vz/template/cache

# Find the most recent backup file for the given container number
backupfile=$(ls -t vzdump-*-$ctnumber-*.tar.gz | head -n1)

# Prompt for new template name
read -p "Enter new template name: " newtemplatename

# Rename the backup file
mv "$backupfile" "$newtemplatename.tar.gz"

echo "Backup created and renamed to $newtemplatename.tar.gz"
