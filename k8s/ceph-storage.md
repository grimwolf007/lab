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
  # In vm with cephadm

# Installation
  # In vm with cephadm

# Configuration

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

# Upgrade Path

# Maintenance

<Other>

# Troubleshooting

# Examples

# Additional Pages

# Additional Resources

# Related Information

### Issue #

<Links>

[1]:
[2]:
[3]:
