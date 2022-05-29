# Proxmox install

## Setup Install Drive
 - Downloaded [Rufus](https://rufus.ie/en/)
 - Downloaded [Proxmox](https://www.proxmox.com/en/downloads/category/iso-images-pve)
 - Using rufus, mounted ISO on 8Gb drive in DD mode

## Raid Controller
 - Setup drives so they appear on the operating system
 - **CAUTION CEPH WILL NOT RUN ON HARDWARE RAID DRIVES**
  - should work if you set each drive to Raid 0 in their own groups (Untested)
## Bios
 - Turned on Virtualization
 - Moved USB to top on boot sequence (move that to the bottom after install)

## Install Proxmox
 - Ensure it is at the right address
   - If not change /etc/network/interfaces and /etc/hosts and restart
 - Disable the upgrade proxmox prompt [link](https://johnscs.com/remove-proxmox51-subscription-notice/)
   - One Liner: `sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service`
   - Then hold [shift] and press the restart button on your browser
# Move to no-subscription repo
`https://pve.proxmox.com/wiki/Package_Repositories`
as root
```
cat << EOF >> /etc/apt/sources.list
# PVE pve-no-subscription repository provided by proxmox.com,
# NOT recommended for production use
deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription
EOF
```
Comment out all lines in `/etc/apt/sources.list.d/pve-enterprise.list`

## Add Second Data Volume to Proxmox
 - Login at https://{pve}:8006
 - On the left sidebar select serverview and click your proxmox server
 - On the inner left sidebar scroll down and select LVM-Thin
 - [Create ThinPool]
 - Select the drive and the size you want to provision

## Setup Firewalls and ipsets [link](https://dannyda.com/2021/10/10/how-to-use-proxmox-ve-pve-firewall-ipset-alias-security-group-etc-basics-about-pve-firewall/)

# Create User Account
 - (Proxmox does not have sudo.)
 - `useradd -U -m john` (
   - `-U` Create user group called {user}
   - `-m` Create home directory /home/{user}
 - `passwd john` 
  - add password
# Add public Key login
- `ssh-keygen` (create an ssh key if you don't have one)
- `ssh-copy-id john@{IP} -i {SSH Key.pub}` (Copy the key over to the other server)
- ```
  cat << EOF >> ~/.ssh/config
  Host pve 
       HostName {IP}
       User john
       PreferredAuthentications publickey
       IdentityFile {SSH Key}
  ```
- `passwd -l john`
  - Sets password as expired (disables password login) 

# Disable root ssh login
 - `vi /etc/ssh/sshd_config`
 - replace `PermitRootLogin yes` with `#PermitRootLogin yes`
