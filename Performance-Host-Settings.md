EdgeFS designed for high performance and massive scalability beyond 1000 servers per single site / namespace physical cluster. It doesn't need to have central metadata server(s) or coordination server(s). Architecture is true "shared nothing" with metadata and data fully distributed across the physical nodes with zoned disks in cluster. While not required, to operate optimally and at highest possible performance, EdgeFS needs dedicated high-performance network for cluster backend communications, isolated with VLAN segment, set for use of Jumbo Frames and preferably non-blocking switch with Flow-Control enabled.

The following are the recommendations you should set on all data nodes where you planning to run "Target" software.

Set optimal host sysctl parameters:

```
#
# Avoid occasional UDP drops and improve networking performance of
# Replicast backend network. It can also be set on per interface basis!
echo "net.core.optmem_max = 131072" >> /etc/sysctl.conf
echo "net.core.netdev_max_backlog = 300000" >> /etc/sysctl.conf
echo "net.core.rmem_default = 80331648" >> /etc/sysctl.conf
echo "net.core.rmem_max = 80331648" >> /etc/sysctl.conf
echo "net.core.wmem_default = 33554432" >> /etc/sysctl.conf
echo "net.core.wmem_max = 50331648" >> /etc/sysctl.conf
#
# Highly desirable to set Linux VM subsytem optimally
echo "vm.dirty_ratio = 10" >> /etc/sysctl.conf
echo "vm.dirty_background_ratio = 5" >> /etc/sysctl.conf
echo "vm.dirty_expire_centisecs = 6000" >> /etc/sysctl.conf
echo "vm.swappiness = 25" >> /etc/sysctl.conf
#
# Improve latency of HTTP-based protocols (EdgeX-S3, S3, S3A, SWIFT)
echo "net.ipv4.tcp_fastopen = 3" >> /etc/sysctl.conf
echo "net.ipv4.tcp_mtu_probing = 1" >> /etc/sysctl.conf
#
# Required for backend IPv6 network
echo "net.ipv6.conf.all.force_mld_version = 1" >> /etc/sysctl.conf
echo "net.ipv6.ip6frag_high_thresh = 10000000" >> /etc/sysctl.conf
echo "net.ipv6.ip6frag_low_thresh = 7000000" >> /etc/sysctl.conf
echo "net.ipv6.ip6frag_time = 120" >> /etc/sysctl.conf
sysctl -p
```
