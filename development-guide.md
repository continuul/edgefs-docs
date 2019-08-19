# Development-Guide

## Development environment

Linux Distributions supported list:

* Ubuntu Server LTS 18.04 x86-64bit \([\#3](https://github.com/highpeakdata/edgefs-docs/tree/89436343de8cea2f3715f31bb26ae96f1641d22b/issues/3/README.md)\)
* Ubuntu Server LTS 16.04 x86-64bit \(default\)
* RedHat Enterprise Linux 7
* CentOS 7

## Coding guide lines

* KISS - Keep It Simple, Stupid
* Coding Style: [https://www.kernel.org/doc/Documentation/CodingStyle](https://www.kernel.org/doc/Documentation/CodingStyle)
* development environment needs to be as close to actual deployment

  environment as possible, i.e. everything is installed under /opt/nedge,

  including build dependencies and production libraries

* most of the development is done with Ubuntu 16.04 since it has advanced

  version of gcc with ASAN included to detect memory leaks

* use top level Makefile and env.sh to provide support for new OSs

## Building \(Development Only\)

1. Install prerequisites GNU make, git and clone project

```text

apt-get install git make
git clone git@github.com:Nexenta/edgefs.git
```

1. Setup target destination, default target location \(controlled by environment variable NEDGE\_HOME\) is /opt/nedge \(can be any\). Use default environment

```text

rm -rf /opt/nedge; /opt/nedge
cd edgefs
source env.sh
```

1. Install development packages \(OS is autodetected\)

```text

make deploy
```

1. Build without ASAN \(production\)

```text

make NEDGE_NDEBUG=1 world
```

Note, we believe that quality and clarity of the code and pull requests makes significant difference in contributions. It saves other's people time, it helps project to achieve needed stability levels and community to progress faster! As such, our default build method selected to always compile with ASAN across all components where possible. So, to enable ASAN, just do not provide NEDGE\_NDEBUG=1 in the command line above. Please make sure that all ASAN found issues needs to be addressed prior to sending pull request!

## Running a test cluster in single node dev environment

1. Ensure that environment set correctly and points to the correct target directory

```text

cd edgefs
source env.sh
```

1. Initialize simple single data directory configuration

```text

mkdir /data
efscli config node -i eth0
```

This will enable use of IPv4 addressing for existing interface eth0 used as a primary backend interface. It will also create 4 subdirectories in /data/device{0,1,2,3} to emulate disks. Consider other profiles to enable more sophisticated configurations, e.g. All-Flash, All HDD, Hybrid HDD with Metadata Offload on SSD, or Mounted Directories.

1. Start Target

```text

edgefs-start.sh target
```

1. Verify that HW \(or better say emulated in this case\) configuration look normal and accept it

```text

efscli system fhtable print
efscli system fhtable set
```

At this point new dynamically discovered configuration checkpoint will be created at $NEDGE\_HOME/var/run/flexhash-checkpoint.json

1. Initialize cluster

```text

efscli system init
```

This will create system "root" object, holding Site's Namespace. Namespace may consist of more then single region.

1. Create new local namespace \(or we also call it "Region" or "Segment"\)

```text

efscli cluster create Hawaii
```

1. Create logical tenants of cluster namespace "Hawaii", also buckets if needed

```text

efscli tenant create Hawaii/Cola
efscli bucket create Hawaii/Cola/bk1
efscli tenant create Hawaii/Pepsi
efscli bucket create Hawaii/Pepsi/bk1
```

Now cluster is setup, services can be now created and attached to CSI provisioner.

## Running unit tests \(Development Only\)

1. Compile unit tests

```text

make NEDGE_NDEBUG=1 test
```

1. Unit tests binaries isn't getting installed into the target directory by default, so, make sure you install it first manually or execute in place. For example core libraries tests can be executed like this:

```text

cd src/ccow/test
./get_test -n
```

Be aware of "-n" flag. It will skip start of Target daemon as a part of the test.

