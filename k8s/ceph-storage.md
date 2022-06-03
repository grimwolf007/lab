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

# Architecture
 - Ceph Monitor (MON)
   - Maintains the cluster map and ensures HA. 
 - Ceph Operating System Disk (OSD) Daemon
   - Reports the state of the disk to ceph MONs and other ceph OSDs
   - Uses the CRUSH algorith to find and store data in the cluster provided by `librados` and other services
 - Ceph Manager
   - The endpoint for monitoring, orchestration and plug-ins 
 - Ceph Metadata Server (MDS)
   - Manages the file metadata for CephFS 

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
