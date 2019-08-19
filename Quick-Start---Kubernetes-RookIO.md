Rook enables EdgeFS storage systems to run on Kubernetes using Kubernetes primitives.

![EdgeFS Rook Architecture on Kubernetes](https://github.com/rook/rook/blob/master/Documentation/media/edgefs-rook.png)

With Rook running in the Kubernetes cluster, Kubernetes PODs or External applications can
mount block devices and filesystems managed by Rook, or can use the S3/Swift API for object storage. The Rook operator
automates configuration of storage components and monitors the cluster to ensure the storage remains available
and healthy.

The Rook operator is a simple container that has all that is needed to bootstrap and monitor the storage cluster. The operator will start and monitor StatefulSet storage Targets, gRPC manager and Prometheus Multi-Tenant Dashboard. All the attached devices (or directories) will provide pooled storage site. Storage sites then can be easily connected with each other as one global name space data fabric. The operator manages CRDs for Targets, Scale-out NFS, Object stores (S3/Swift), and iSCSI volumes by initializing the pods and other artifacts necessary to
run the services.

The operator will monitor the storage Targets to ensure the cluster is healthy. EdgeFS will dynamically handle services failover, and other adjustments that maybe made as the cluster grows or shrinks.

The EdgeFS Rook operator also comes with tightly integrated CSI plugin. CSI pods deployed on every Kubernetes node. All storage operations required on the node are handled such as attaching network storage devices, mounting NFS exports, and dynamic provisioning.

Rook is implemented in golang. EdgeFS is implemented in C/Go where the data path is highly optimized.
While fully immutable, you will be impressed with additional capabilities EdgeFS provides besides ultra high-performant storage, i.e. built in Data Reduction with Global De-duplication and on the fly compression, at rest encryption and per-Tenant QoS controls.

Detailed instructions on how to get EdgeFS Rook operator configured [available at Rook.IO web site](https://rook.github.io/docs/rook/master/edgefs-quickstart.html)

## Advanced Configuration: Multi-Segment Cluster

With Rook EdgeFS operator we can configure 2+ segments within same Kubernetes cluster installation. This can be useful when Kubernetes nodes span out across multiple regions and cross-region latency can be high, or links can be temporarily offline.

EdgeFS ISGW Links can be setup to ensure consistent synchronization of all segments on per-bucket basis.

Each Segment has to have its own region sub-namespace, local to region tenants and its users. For example, two syncing segments of the same global namespace can be seen in `efscli` as this:

```
# efscli cluster list
SanFrancisco
NewYork

# efscli tenant list SanFrancisco
Biology
MedicalSupply

# efscli tenant list NewYork
Marketing
Finance
```

### Configuring Segments
Ensure that each Segment is configured on per its own Kubernetes namespace. For that, copy cluster.yaml CRD file and modify all occurrences of:

* Namespace name
* PodSecurityPolicy metadata name
* ClusterRole metadata name
* ClusterRoleBinding system-psp and cluster-psp metadata name and roleRef
* namespace metadata

Create new cluster CRD and observe that it will be created in its own namespace. Pay attention to filtering node and device selectors. Node's devices cannot be shared between namespaces.

The end result may look like this:

```
# kubectl get svc -n rook-edgefs-sanfrancisco
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
rook-edgefs-mgr       ClusterIP   10.110.194.251   <none>        6789/TCP                        2m59s
rook-edgefs-restapi   ClusterIP   10.110.211.133   <none>        8881/TCP,8080/TCP,4443/TCP      2m59s
rook-edgefs-target    ClusterIP   None             <none>        5405/UDP                        2m59s
rook-edgefs-ui        NodePort    10.96.209.62     <none>        3000:31718/TCP,3443:31170/TCP   2m59s
# kubectl get svc -n rook-edgefs-newyork
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
rook-edgefs-mgr       ClusterIP   10.104.208.186   <none>        6789/TCP                        89s
rook-edgefs-restapi   ClusterIP   10.109.33.212    <none>        8881/TCP,8080/TCP,4443/TCP      89s
rook-edgefs-target    ClusterIP   None             <none>        5405/UDP                        89s
rook-edgefs-ui        NodePort    10.100.223.220   <none>        3000:31324/TCP,3443:31937/TCP   89s
# kubectl get svc -n rook-edgefs-sandiego
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
rook-edgefs-mgr       ClusterIP   10.99.38.109    <none>        6789/TCP                        25s
rook-edgefs-restapi   ClusterIP   10.96.247.151   <none>        8881/TCP,8080/TCP,4443/TCP      25s
rook-edgefs-target    ClusterIP   None            <none>        5405/UDP                        25s
rook-edgefs-ui        NodePort    10.103.27.110   <none>        3000:32720/TCP,3443:30906/TCP   25s
```

Where each cluster segment has its own management end points, and yet controlled by the same Rook Operator instance.

### Configuring Services

After all EdgeFS Targets of a new segment are up and running, verify that EdgeFS UI can be accessed and it can create Services CRDs. While creating services, it would automatically pick up Kubernetes operating namespace and provide it for a CRD's metadata.

Similarly, if you prefer to operate cluster segments via CLI, you can use neadm management command: `neadm service enable|disable NAME` that will create/delete CRD similarly to GUI using same REST API calls.

Finally, alternative way is to manually prepare CRD YAML file as per instructions on [Rook documentation website](https://rook.github.io/docs/rook/master/edgefs-storage.html) and specify target Kubernetes namespace.

### Configuring CSI provisioner

TBD.

At the moment, CSI Topology Awareness is still in Alpha status and as such EdgeFS CSI driver does not support it just yet:

[CSI Topology Awareness](https://kubernetes-csi.github.io/docs/topology.html)

The default logic will apply, that is volume name will be using global matching pattern.
