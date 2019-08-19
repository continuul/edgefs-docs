## Setting up FlexHash and Site root object

Before new local namespace (or local site installation) can be used, it has to be initialized with FlexHash and special purpose root object.

FlexHash consists of dynamically discovered configuration and checkpoint of accepted distribution table. FlexHash is responsible for I/O direction and plays important role in dynamic load balancing logic. It defines so-called Negotiating Groups (typically formed across zoned 8-24 disks) and final table distribution across all the participating components, e.g. data nodes, service gateways, and tools.

Root object holds system information and table of cluster namespaces registered to a local site. Root object is always local and never shared between the sites.

Assumption at this point is that nodes are all configured and can be seen via the following commands:

<pre>
efscli system status
efscli system status -v1
</pre>

1. Initialize cluster

<pre>
efscli system init
</pre>

This will create system "root" object, holding site's cluster namespaces. A global namespace may consist of a more than a single geographical region.

2. Create new local namespace (or we also call it "Region" or "Segment")

<pre>
efscli cluster create Hawaii
</pre>

3. Create logical tenants of cluster namespace "Hawaii", also buckets if needed

<pre>
efscli tenant create Hawaii/Cola
efscli bucket create Hawaii/Cola/bk1
efscli tenant create Hawaii/Pepsi
efscli bucket create Hawaii/Pepsi/bk1
</pre>

Now cluster is setup, services can be now created and attached to CSI provisioner or continue with specific [configurations](https://github.com/Nexenta/edgefs/wiki/Quick-Start-Configurations)