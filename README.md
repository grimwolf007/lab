# The Homelab

# Goals
 - Create a virtual business network to run viruses and test exploits in
 - Be able to automatically stand up infrastructure for my game
 - Create tools that would be useful for me
   - gitlab ci/cd
   - Kubernetes cluster
 - Learn how different infrastucture works without risk of damaging anything

# Block yourself off
If you are going to be learning hacking, or playing with viruses in a sandbox make sure to block yourself off from the world.
## Firewall
An easy way to do this is to start with a firewall and section off different parts of the network.
### Network segments
 - I suggest 3 subnets (but I do not speciallize in networking, this is just what has been working for me)
  - Blue team net: A sandbox with internet where this is nothing bad. It allows you to learn about new software, how to harden things, or play around with networking.
  - Red team net: A place where you launch scans at the blue team, or attack other systems on the same red team network. For this network you only have the things you need on at a time. You block prepare the firewall for what you may work on, so it forces your tools or the malware to stay in scope. You might even have a DNS Black Hole set up, so if it tries to call to a domain it will receive a fake IP. 
  - Gold team net: A place that is out of bounds when it comes to hacking and scaning scope. This is where you have your internet proxy, documentation, mail forwarder, DNS forwarder etc... for the other networks. 

 - You can also do what small bussinesses do
  - Private network: Network dedicated to Business tasks that do not need customer access. HR DB, Internal Email, Internal Chat, Print servers, Employee computers... 
  - Public network: Network dedicated to inward traffic from the internet. Monitored well and isolated from the private network. Website, Customer Applications...
  - Demilitarized Zone (DMZ): Network dedicated to anything that goes between the private and public networks, or private and internet. Internet proxy, public email server, DNS... 

 - Almost every hobbiest lab will be built on a hypervisor so you can use VMs to make whatever hardware you want, and tear it down when you need to. 
  - Make backups to external "cold" drives (not normally used so if something attacks your network it can't survive in them) 
  - Keep your systems monitored for malware.
  - Make sure you are monitoring the hardware for failures.
  - Be ready to pull the plug if you need to. You are not a company. You are learning. When things are going bad know when to stop and get help if you need it.


# Basic Lab
You don't need a fancy server to test things out.
Don't forget that the best way to learn is to struggle and mess things up. Build applications from github. Add features to them. Install things wrong. Simulate disasters. Roll back from installs. Flicker the internet. Send wrong data. Read logs. Read CVEs. Install really old software, use it for a week and update it to the most recent version and see if it survived. Have fun!


## [Virtualbox](https://www.virtualbox.org/)
- The simplist way I have found is to use virtualbox.
- It is a software Hypervisor (way to run virtual machines)
### Install

### Setup (Networking)

### First VM (kali)
- https://www.kali.org/get-kali/#kali-virtual-machines
- Kali is an environment built on debian that has a bunch of penatration testing tools already installed.
### Second VM (metasploitable)
- https://information.rapid7.com/metasploitable-download.html
- Metasploitable 2 is a vulnerable ubuntu linux VM that is built to test metasploit (A popular free penetration testing tool).
- https://docs.rapid7.com/metasploit/metasploitable-2-exploitability-guide/
### Third VM (Ubuntu Desktop)
- https://ubuntu.com/download/desktop
- Ubuntu is a fairly user friendly distribution of linux
- https://overthewire.org/wargames/
- This website is good to learn the bash command line, which is a very import skill for using linux in general
### Fourth VM (Ubuntu Server)
- Once you get the hang of command line you can choose to abanton a gui all together.
- Realisticly this is to allow you to run a service where you want to dedicate as many resouces as possible to it. GUIs use a ton of RAM and CPU cycles, and all those pictures need to be saved somewhere.

### Learn Networking (PFSense)
- Pfsense is a free and powerful router software with a helpful community
- Make some firewall rules
- See what traffic is passing though your network with tcp-dump

# Windows 
- powershell is a good tool to learn 

# Linux
- VIM https://vim-adventures.com/

# Networking
- https://github.com/errorinn/netsim ?
