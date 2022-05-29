# Proxmox setup
 - Cpus: 1
 - ram: 1024
 - networks:
   - vmbr0 : WAN
   - vmbr1 : Gold
   - vmbr2 : kube
//TODO   - vmbr3 : Red/blue
 - Display: Spice

# PFsense Setup
 - VMnet0: WAN
 - VMnet1: LAN (GOLD)

# [Squid proxy Setup](https://kifarunix.com/install-and-setup-squid-proxy-on-pfsense/)
 - 

# Change admin password
 - Settings -> User Manager -> Pencil on Admin -> Add new passwords -> Save

# Setup OPT1
 - Add interface
 - Ensure TCP, UDP and ICMP are all allowed in firewall

# Create VLAN
//TODO
Red/blue
