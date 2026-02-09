# WSL Oracle Linux 9 Installation Guide

## Prerequisites

Open PowerShell in administrator mode and update WSL:
```powershell
wsl --update
```

## Step 1: Install Oracle Linux

Install Oracle Linux 9.5. After the distro installs, it will ask you to create a user. Make sure to note down the password.
```powershell
wsl --install -d OracleLinux_9_5
```

## Step 2: Configure WSL Settings

In Oracle Linux, create `/etc/wsl.conf` and append WSL settings. Switch to root user first:
```bash
sudo su - root
```

Create the configuration file:
```bash
{
echo "[network]"
echo "generateResolvConf = false"
echo ""
echo "[user]"
echo "default=root"
echo ""
echo "[boot]"
echo "systemd=true"
} >> /etc/wsl.conf
```

## Step 3: Restart WSL

Shutdown WSL in PowerShell, then open the VM again via the "Oracle Linux X.X" Windows app:
```powershell 
wsl --shutdown
```

## Step 4: Update System

Update the VM. Switch to root user first:
```bash
sudo su - root
```

Run system updates and install required packages:
```bash
echo "nameserver 8.8.8.8" > /etc/resolv.conf && \
echo "nameserver 8.8.4.4" >> /etc/resolv.conf && \
dnf upgrade -y && \
dnf install -y hostname zip unzip cronie ncurses expect wget curl git vim nano nc crypto-policies-scripts && \
update-crypto-policies --set LEGACY && \
systemctl enable crond && \
systemctl start crond && \
dnf clean all
```
