# Ceph Cluster
- Ceph is a Storage cluster that provides:
   - Object storage: S3 compatible storage that holds files as objects with associated metadata in buckets.
   - Block storage: Breaks a file into blocks that are then stored at different addresses.
   - Filesystem storage: What a normal person works in with where files are held in folders/directories similar to a paper filing system.


# Table of Contents
* [System Requirements](#system-requirements)
* [Installation](#installation)
* [Configuration](#configuration)
* [Architecture](#architecture)
* [Upgrade Path](#upgrade-path)
* [Maintenance](#maintenance)
* [Troubleshooting](#troubleshooting)
* [Examples](#examples)
* [Additional Pages](#additional-pages)
* [Additional Resources](#additional-resources)
* [Related Information](#related-information)

# System Requirements
## In vm with cephadm (automatically installed via bootstrap)
 - Python3
 - Systemd
 - Podman or docker (podman recommended)
 - Time syncronization (Chrony or NTP)
 - LVM2

# Installation
## In vm with cephadm
- [ ] Install cephadm
  - [ ] Curl 
     - `curl --silent --remote-name --location https://github.com/ceph/ceph/raw/quincy/src/cephadm/cephadm`
     - `chmod +x cephadm`
     - `./cephadm add-repo --release quincy`
     - `./cephadm install`
     -  Confirm install `which cephadm` Output: /usr/sbin/cephadm
  -  [ ] Centos Stream
     - `dnf search release-ceph`
     - `dnf install --assumeyes centos-release-ceph-quincy`
     - `dnf install --assumeyes cephadm`
 - [ ] [Bootstrap](https://docs.ceph.com/en/quincy/cephadm/install/#running-the-bootstrap-command)
   - `cephadm bootstrap --mon-ip <mon-ip>`
   - additional options are available  
 - [ ] Add hosts 
   > Cephadm must be able to log into all the Ceph cluster nodes as an user that has enough privileges to download container images, start containers and execute commands without prompting for a password. If you do not want to use the “root” user (default option in cephadm), you must provide cephadm the name of the user that is going to be used to perform all the cephadm operations. Use the command: 
   ```
   ceph cephadm set-user <user>
   ```
   > Prior to running this the cluster ssh key needs to be added to this users authorized_keys file and non-root users must have passwordless sudo access.

  
   - For each new host, from the master 
     - `ssh-copy-id -f -i /etc/ceph/ceph.pub root@<new-host>` 
     - `ceph orch host add <newhost> <ip>` without IP, it will be resolved with DNS
       - if `--labels _admin` is added it will maintain a copy of the cluster config and keyring.
         - Labels only allow you to choose where to put daemons. They are more for comments about hosts.
       - [ ] For many hosts:
         ```
         cat << EOF >> cluster-inventory.yaml
         service_type: host
         hostname: node-00
         addr: 192.168.0.10
         labels:
         - example1
         - example2
         ---
         service_type: host
         hostname: node-01
         addr: 192.168.0.11
         labels:
         - grafana
         ---
         service_type: host
         hostname: node-02
         addr: 192.168.0.12
         EOF
         ```
# Configuration
 - [CephFS](https://docs.ceph.com/en/latest/cephadm/services/mds/#orchestrator-cli-cephfs) `ceph fs volume create <fs name>`
   - Should Create MDSs automatically, if not you can deploy them [manually](https://docs.ceph.com/en/quincy/cephfs/add-remove-mds/)
 - [Ceph Object Gateway](https://docs.ceph.com/en/latest/cephadm/services/rgw/#cephadm-deploy-rgw) ``
 - [Ceph Multi-Site (GEO-Replication)](https://docs.ceph.com/en/quincy/radosgw/multisite/)

# [Architecture](https://docs.ceph.com/en/quincy/architecture/)

## Components
- Ceph Monitor (MON)
   - Maintains the cluster map and ensures HA. 
 - Ceph Operating System Disk (OSD) Daemon
   - Reports the state of the disk to ceph MONs and other ceph OSDs
   - Uses the CRUSH algorith to find and store data in the cluster provided by `librados` and other services
 - Ceph Manager
   - The endpoint for monitoring, orchestration and plug-ins 
 - Ceph Metadata Server (MDS)
   - Manages the file metadata for CephFS 

## Storing Data
 - Data is recieved from a ceph client and goes through the Block Device, Object Storage or File System API Where is is stored as RADOS object on an XFS filesystem. 
 - Ceph also supports data pools which can limit storage for applications and change replication rules for data. They require the following: Who owns the data and who has access, number of placement groups (replicas), and which CRUSH rules to use

## [Cluster Maps](https://docs.ceph.com/en/quincy/architecture/#cluster-map)
Each map contains the status and info about the ceph components
- Monitor: `ceph mon dump`
 - OSD: `ceph osd dump`
 - Placement Group (PG) `ceph pg dump`
 - CRUSH: `ceph osd getcrushmap -o /tmp/CRUSH.map.comp; crushtool -d /tmp/CRUSH.map.comp -o /tmp/CRUSH.map; cat /tmp/CRUSH.map; rm /tmp/CRUSH.ma*;`
 - Metadata Server (MDS): `ceph fs dump`

## Cluster Quorum
 - Monitor Majority and [Paxos](https://en.wikipedia.org/wiki/Paxos_(computer_science))
   - Paxos garuntees Validity and Agreement, not Termination 
     - Only valid data is stored. Data will not diverge. The algorithm may loop.

## Authentication
  - Uses cephx (similar to Kerberos), but all monitors can generate tickets.
  - Cephx does not encrypt data in transit or at rest

# [Upgrade Path](https://docs.ceph.com/en/quincy/cephadm/upgrade/)
 - [ ] Ensure health of cluster `ceph -s`
 - [ ] `ceph orch upgrade start --ceph-version <version>`
   - Dockerhub is no longer used, make sure it points to the correct container `ceph orch upgrade start --image quay.io/ceph/ceph:v16.2.6`
 - [ ] Monitor the upgrade `ceph orch upgrade status`
   - For a progress bar `ceph -s`
   - For the log `ceph -W cephadm`
 - **To stop the upgrade** `ceph orch upgrade stop`
## Common issues
### Not enough managers
  - To see existing `ceph orch ps --daemon-type mgr`
  - To restart stopped `ceph orch daemon restart <name>`
  - To make more `ceph orch apply mgr 2  # or more`
### Failed to pull
  - Make sure you are using the right image from the right repo
  - Ensure each node can pull the image
  - `ceph orch upgrade stop`
  - `ceph orch upgrade start --ceph-version <version>`
# Maintenance
## Start OSD
  - When an OSD is added it creates a service. 
  - [ ] Start it 'sudo systemctl start ceph-osd@{osd-num}'
  - [ ] Enable it 'sudo systemctl enable ceph-osd@{osd-num}'
## Add a new OSD (Expand Capacity)
 - Ceph defaults to One Daemon per drive, if you have many drives on one host you can have one daemon manage many drives
 - Weight is for adding drives with differing sizes
 - [ ] On the new host
   - [ ] Format the drive `sudo mkfs -t xfs /dev/{drive}`
   - [ ] Get the UUID `lsblk -f`
 - [ ] On master `ceph osd create {UUID}` save the ID
 - [ ] On the new host 
   - [ ] Mount the drive `sudo mount -o user_xattr /dev/{drive} /var/lib/ceph/osd/ceph-{ID}`
   - [ ] Initialize the OSD `ceph-osd -i {ID} --mkfs --mkkey
   - [ ] Register the auth key `ceph auth add osd.{id} osd 'allow *' mon 'allow rwx' -i /var/lib/ceph/osd/ceph-{id}/keyring`
   - [ ] Add the OSD to the crush map `ceph osd crush add {ID} {weight}`  

## Replace OSD (Disk Failure)
  - [ ] Wait for osd to be safe for removal `while ! ceph osd safe-to-destroy osd.{id} ; do sleep 10 ; done
`
  - [ ] Destroy the OSD `ceph osd destroy {id} --yes-i-really-mean-it`
  - [ ] If the replacement Disk was used before, zap it `ceph-volume lvm zap /dev/sdX`
  - [ ] Recreate with old ID `ceph-volume lvm create --osd-id {id} --data /dev/sdX`

## Remove OSD
 - [ ] Remove from cluster `ceph osd out {osd-num}`
 - [ ] Wait for the pg states to change from `active degraded` to `active clean`. `ceph -w`

   **If the PGs get stuck in active+remapped**
   ```
   ceph osd in {osd-num}
   ceph osd crush reweight osd.{osd-num} 0
   ```
  - [ ] Stop the OSD (From the host mounting the OSD) `sudo systemctl stop ceph-osd@{osd-num}` `sudo systemctl disable ceph-osd@{osd-num}`
  - [ ] Remove drive, Daemon and authentication key from cluster `ceph osd purge {id} --yes-i-really-mean-it`
  - [ ] Remove from ceph.conf `vim /etc/ceph/ceph.conf`
  - [ ] Copy the new ceph.conf to the other hosts


# Troubleshooting
 ## To Remove Hosts 
  - [ ] If a host is unrecoverable `ceph orch host rm <host> --offline --force`
  - [ ] `ceph orch host drain <host>`
  - [ ] Ensure all OSDs are removed `ceph orch osd rm status`
  - [ ] Enesure all Daemons are removed `ceph orch ps <host>`
  - [ ] Remove the host `ceph orch host rm <host>`
  

# Examples

# Additional Pages

# Additional Resources

# Related Information

### Issue #

<Links>

[1]:
[2]:
[3]:
