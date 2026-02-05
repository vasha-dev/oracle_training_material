Open powershell in administrator mode, update WSL.
```poweshell
wsl --update
```
1. Install oracle linux. After distro installs. It will ask you to create an user, make sure to mark down the password of the user. 
```powershell
wsl --install -d OracleLinux_9_5
```
2. In oracle linux: Create /etc/wsl.conf and append echo WSL settings. Under root user.  
```SHELL
sudo su - root
```
```SHELL
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
3. Shutdown WSL in powershell. And open VM again via "Oracle Linux X.X" windows app. 
```powershell 
wsl --shutdown
```
4. Update VM. Under root user. 
```SHELL
sudo su - root
```
```SHELL
echo "nameserver 8.8.8.8" > /etc/resolv.conf && \
echo "nameserver 8.8.4.4" >> /etc/resolv.conf && \
dnf upgrade -y && \
dnf install -y hostname zip unzip cronie ncurses expect wget curl git vim nano nc crypto-policies-scripts && \
update-crypto-policies --set LEGACY && \
systemctl enable crond && \
systemctl start crond && \
dnf clean all
```
