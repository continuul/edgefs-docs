# Docker-NFS-service-with-static-exports

A configuration of EdgeFS NFS service needs tenant object to be available. For static configurations, one or more buckets needs to exist and has to be explicitly added to the service object.

1. Let's create service with the name `nfsCola`:

```text

efscli service create nfs nfsCola
```

1. And now "serve" `Hawaii/Cola/bk1` bucket so that it will appear as nfsCola service export:

```text

efscli service serve nfsCola Hawaii/Cola/bk1
```

You can add/remove as many exports as needed prior to enabling service.

1. And finally, let's enable nfsCola service:

```text

docker run -d -v /edgefs/var/run:/opt/nedge/var/run:z \
       -v /edgefs/var/log:/opt/nedge/var/log:z \
       -v /edgefs/etc:/opt/nedge/etc.target:z \
       -e CCOW_SVCNAME=nfsCola \
       edgefs/edgefs:latest nfs
```

EdgeFS NFS service docker container needs access to some local node target's /edgefs/{var,etc} directories.

Bind mounting of following directories enables:

```text

        `/edgefs/etc`     : Access to the target's /opt/nedge/etc, must be bind-mounted as /opt/nedge/etc.target.
        `/edgefs/var/log` : Common log files location shared with target and other EdgeFS services running on the host.
        `/edgefs/var/run` : Access to the target's /opt/nedge/run.
```

Where:

| Option | Notes |
| :--- | :--- |
| -e CCOW\_SVCNAME=name | Required parameter which specifies EdgeFS existing service name |

1. At this point, NFS service should be available and Cola/bk1 export accessible. Verify that showmount command can see service and lists exports:

```text

showmount -e CONTAINERIP
```

