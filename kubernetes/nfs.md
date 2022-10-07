# NFS

Challenge: run mysql-container with /var/lib/mysql located on an NFS-share.

Use NFS4.2, do not use any NFSv3 otherwise error: unable to lock ./ibdata1\
to observe the error:\
`kubectl logs pod_name`\
And mysql won’t even allow you to connect to server from the container shell with message: unable to connect to server via socket.

## NFS version 3 vs NFS vers 4.2

```
minikube ssh
mount | grep nfs | grep “vers=3”
```

In case of non-empty output work on fixing it. Below is the list of issues I hit in the past:

1\)      Issue: insecure disabled. This prevents client form using src UDP ports above 1024. To fix it add insecure flag to NFS-servers share

2\)      Issue: insecure\_lock disabled. This prevents from client to lock the file on nfs-share over connection with src port higher than 1024

3\)      Issue: parent folder of a share has an owner other than nobody:nogroup.\
For example: on a client `mount -t nfs 127.0.0.1:/storage_local/www.connme.ru/logs xxxxxx` in this case owner of the folder /storage\_local/www.connme.ru/ on the server must be nobody:nogroup.

## Refresh shares list on server

```
exportfs -ra
```

## NFS troubleshooting

List of exportable shares from NFS-server

```
showmount -e 192.168.168.32
```

This command expects open communication over tcp/111 and tcp/2904. Expected output:

```
Export list for 192.168.168.32:
/mnt/HD/HD_a2/ikuchin *
/mnt/HD/HD_a2/Public  *
```

#### &#x20;Insecure share

If root share has different flags (in particular missing nosecure), won’t allow successfully run mysql-server with error message “permission denied”\
To [troubleshoot](https://wiki.archlinux.org/title/NFS/Troubleshooting#mount.nfs:\_Operation\_not\_permitted) it on the NFS-server:\
1\) `rpcdebug -m nfsd -s all` (on a server)\
2\) `rpcdebug -m nfs -s all` (on a client)\
3\) `journalctl -fl`\
4\) run successful operation and unsuccessful -> compare the two\
&#x20;    in my case difference is\
\
nfsd: fh\_compose(exp fd:00/1048578 <mark style="background-color:yellow;">/storage\_local</mark>, ino=1048578)\
nfsd: fh\_verify(8: 00010001 00000000 00000000 00000000 00000000 00000000)\
nfsd: request from <mark style="background-color:yellow;">insecure port 192.168.170.35</mark>, port=60424!\
nfsv4 <mark style="background-color:yellow;">compound returned 1</mark>

\
it points out to root NFS-share and issue with insecure port , which must be fixed by configuring “insecure” option on NFS-share.

#### To avoid strange errors – don’t use hierarchy.

Even with previous item – fixed.  \
MySQL server throw a error: “Permission denied” after restarting pod, which is super weird.\
By removing share-hierarchy everything back on track.

## NFS-share access rights

#### Problem 1:&#x20;

* use `root_squash`
* change file ownersip from client&#x20;

{% hint style="info" %}
Impossible by definition of NFS-security.

The only root user can change file owner, therefore root\_squash must be turned of.
{% endhint %}

#### Problem 2:

fsGroup doesn't work on NFS-share&#x20;

Pod spec -> securityContext -> fsGroup makes all files on volume to have groupID equal to what configured in fsGroup. More details [here](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

{% hint style="warning" %}
NFS volumes don't support it.&#x20;
{% endhint %}

#### Scenario 1:

It is known upfront that all files on that share will be accessed by user www-data (33).

Solution 1.1:&#x20;

Make NFS-share owner www-data (33) and enable `root_squash`

Solution 1.2:

Use init container to change share owner&#x20;

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine-efs-1
  labels:
    name: alpine
spec:
  volumes:
  - name: nfs-test
    nfs:
      server: fs-xxxxxxxx.efs.us-east-1.amazonaws.com
      path: /
  securityContext:
    fsGroup: 100
    runAsGroup: 100
    runAsUser: 405
  initContainers:
    - name: nfs-fixer
      image: alpine
      securityContext:
        runAsUser: 0
      volumeMounts:
      - name: nfs-test
        mountPath: /nfs
      command:
      - sh
      - -c
      - (chmod 0775 /nfs; chgrp 100 /nfs)
  containers:
  - name: alpine
    image: alpine
    volumeMounts:
      - name: nfs-test
        mountPath: /nfs
    command:
      - tail
      - -f
      - /dev/null
```

#### Scenario 2

Container must be run with user root (if it runs web-server, socket binding) -> container creates multiple files on a share -> changes files ownership

Solution 2:

Because of the file ownership change, NFS-share must turn off `root_squash`

``
