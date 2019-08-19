# Quick-Start---Docker

## EdgeFS Docker container

Although Setting up a EdgeFS environment is a pretty simple and straight forward procedure, community do maintain docker base image in the docker hub for the ease of use.

The following are the steps to run the EdgeFS docker images:

To pull the docker image from the docker hub run the following command:

```text

docker pull edgefs/edgefs:latest
```

This will pull the EdgeFS docker image from the docker hub. Alternatively, one could build the image from the Dockerfile directly as shown on the main page.

Once the image is built or pulled, ensure the following directories are created on the host where docker is running:

* /edgefs/etc
* /edgefs/var/run
* /edgefs/var/log
* /edgefs/data

Ensure all the above directories are empty to avoid any conflicts.

For multi host deployment, ntp service like chronyd / ntpd service needs to be started in the host. This way all the EdgeFS containers started will be time synchronized.

Time synchronization isn't required but highly recommended as it will keep file / object modification times correlated when multi-head deployment is used.

## EdgeFS single node "Solo" all-in-one mode

"Solo" mode is really just one-liner below to get you up and running in no time. While it may seem it is not much usable, in fact, this simple configuration can serve a purpose of Namespace synchronization between many sites.

Run the following commands:

```text

mkdir -p /edgefs/var/run /edgefs/var/log /edgefs/data /edgefs/etc
docker run -d -v /edgefs/var/run:/opt/nedge/var/run:z \
              -v /edgefs/var/log:/opt/nedge/var/log:z \
              -v /edgefs/etc:/opt/nedge/etc:z \
              -v /edgefs/data:/data:z \
              edgefs/edgefs:latest solo
```

Bind mounting of following directories enables:

```text

        `/edgefs/etc`     : To make configuration persistent in the host.
        `/edgefs/var/log` : To make log files persistent in the host.
        `/edgefs/var/run` : To make state files persistent in the host.
        `/edgefs/data`    : To make metadata and data persistent in the host.
```

It will use eth0 networking device by default inside of the container and create 4 emulated devices in /edgefs/data/device-{0,1,2,3}. If you want to use different networking interface pass environment variable like this "-e NEDGE\_SOLOIF=eth1".

Notice that in case of forceful container termination \(docker rm -f\) /edgefs/var/run/\*.pid files may prevent container from starting again. Clean up \*.pid files if that happens.

At this point, a single node cluster should be formed and ready to be initialized.

You will need to login into the container in terms of to get access to `efscli` administration tool. You can use `toolbox` command:

```text

docker exec -it CONTAINERID toolbox
```

Follow [Quick Start on Initialization](https://github.com/Nexenta/edgefs/wiki/Quick-Start---Initialization) to continue with setting it up and later bringing up services you need.

## EdgeFS multi-node setup, high-performance

Solo mode is simple but it is limited to a single node and cannot provide availability at the host or zone level. Solo mode is perfect for embedded caching access points. However, if you want to build high-performant setup with enhanced availability, it needs to be networked.

Before forming the cluster you need to decide on networking topology. For simplistic deployments, we recommend a single flat network with IPv4 and no Multicast. Below is an example on how to get it setup.

### Init local data nodes

The first step is to initialize local node networking and storage device configurations.

For networking, let's assume we have 3 nodes connected over flat IPv4 protocol. A number of nodes have to be 3 or more in terms of to provide replication count=3 level availability across 3 nodes. That is cluster can sustain a loss of any of 2 nodes. This is default cluster namespace configuration and assumption. If you planning to use 2 node cluster, logical namespace needs to be created with `--replication-count=2` option.

For storage devices, you need to decide what you want to use for persistent storage. EdgeFS supports local filesystem directories and raw disks. When building high-performance configurations, usage of raw disks is highly recommended.

#### Case 1: Utilizing raw disks

To enable usage of raw disks, they need to be completely empty, without partitions or filesystems. Use `wipefs -a /dev/DEV` command to clear them up. EdgeFS node config tool will autodetect available disks and enable optimal configuration to use. Run the following interactive command on all the nodes which you want to be part of cluster and serve HDDs or SSD/NVMe as raw disks:

```text

docker run --rm -it -v /edgefs/var/run:/opt/nedge/var/run:z  \
              -v /edgefs/var/log:/opt/nedge/var/log:z \
              -v /edgefs/etc:/opt/nedge/etc:z \
              -v /run/udev:/run/udev:ro \
              --privileged=true --net=host -v /dev/:/dev \
              edgefs/edgefs:latest config \
                  -i eth0 -d rtrd -p rtrdMDOffload \
                  -l node1 -l node2 -l node3
```

#### Case 2: Utilizing local directories

Run the following interactive command on all the nodes which you want to be part of a cluster serving /edgefs/data directory:

```text

docker run --rm -it -v /edgefs/var/run:/opt/nedge/var/run:z  \
              -v /edgefs/var/log:/opt/nedge/var/log:z \
              -v /edgefs/etc:/opt/nedge/etc:z \
              -v /run/udev:/run/udev:ro \
              -v /edgefs/data:/data:z \
              --privileged=true --net=host -v /dev/:/dev \
              edgefs/edgefs:latest config \
                  -i eth0 -d rtlfs \
                  -l node1 -l node2 -l node3
```

Where \(both cases options aggregated\):

| Option | Notes |
| :--- | :--- |
| --net=host | This option brings maximum network throughput for your storage container |
| --privileged=true | If you are exposing the `/dev/` tree of host to the container to use raw disks |
| -i eth0 | Networking interface to use for backend I/O |
| -d rtrd | Selecting RTRD driver, e.g. Raw Disk w/o the need for Filesystem, or rtlfs for directories |
| -p rtrdMDOffload | Enable hybrid HDD with Metadata on SSD profile, groups will be autodetected |
| -l node1 -l ... | List of data nodes which will be part of the local cluster namespace \(site\). Can be IPv4 addresses or resolvable hostnames of interfaces where FlexHash needs to be communicating on. Typically colocated with backend networking device. |

Bind mounting of following directories enables:

```text

        `/edgefs/etc`     : To make configuration persistent in the host.
        `/edgefs/var/log` : To make log files persistent in the host.
        `/edgefs/var/run` : To make state files persistent in the host.
        `/edgefs/data`    : In case of `rtlfs` use this host location as emulated storage devices.
        `/run/udev`       : To access extended device attributes via udevadm.
```

### Connect data nodes into the cluster

Data nodes are the nodes which will be serving data. We call it targets. Data nodes need to run network-connected "Target" software in terms of to form the local site cluster.

For all data nodes, to get optimal performance please apply the appropriate [Performance Host settings](https://github.com/Nexenta/edgefs/wiki/Performance-Host-Settings).

Now that all the nodes successfully initialized, run the following command on all then nodes where you want to run "Target" software:

```text

docker run -d -v /edgefs/var/run:/opt/nedge/var/run:z  \
              -v /edgefs/var/log:/opt/nedge/var/log:z \
              -v /edgefs/etc:/opt/nedge/etc:z \
              -v /edgefs/data:/data:z \
              --privileged=true --net=host -v /dev/:/dev \
              edgefs/edgefs:latest target
```

At this point, the cluster should be formed and ready to be initialized.

You will need to login into the container in terms of to get access to `efscli` administration tool. You can use `toolbox` command:

```text

docker exec -it CONTAINERID toolbox
```

Follow [Quick Start on Initialization](https://github.com/Nexenta/edgefs/wiki/Quick-Start---Initialization) to continue with setting it up and later bringing up services you need.

