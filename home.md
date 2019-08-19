# EdgeFS - a multi-cloud scalable distributed storage system

EdgeFS is high-performance and low-latency object storage system released under Apache License v2.0 developed in C/Go. It provides Kubernetes integrated Multi-Head Scale-Out NFS \(POSIX compliant, Distributed RW access to files\), Amazon S3 compatible API, iSCSI or NBD block interfaces, advanced global versioning with file-level granularity unlimited snapshots, global data deduplication and Geo-transparent access to data from on-prem, private/public clouds or small footprint edge \(IoT\) devices.

![edgefs-multicloud.png](https://github.com/Nexenta/edge-dev/raw/master/images/edgefs-multicloud.png?raw=true)

EdgeFS is capable of spanning unlimited number of geographically distributed sites \(Geo-site\), connected with each other as one global name space data fabric running on top of Kubernetes platform, providing persistent, fault-tolerant and high-performance volumes for stateful Kubernetes Applications.

At each Geo-site, EdgeFS nodes deployed as containers \(StatefulSet\) on physical or virtual Kubernetes nodes, pooling available storage capacity and presenting it via compatible S3/NFS/iSCSI/etc storage emulated protocols for cloud-native applications running on the same or dedicated servers.

## How it works, in a Nutshell?

If you familiar with "git", where all modifications are fully versioned and globally immutable, it is highly likely you already know how it works at its core. Think of it as a world-scale copy-on-write technique. Now, if we can make a parallel for you to understand it better - what EdgeFS does, it expands "git" paradigm to object storage and making Kubernetes Persistent Volumes accessible via emulated storage standard protocols e.g. S3, NFS and even block devices such as iSCSI, in a high-performance and low-latency ways. With fully versioned modifications, fully immutable metadata and data, users data can be transparently replicated, distributed and dynamically pre-fetched across many Geo-sites.

