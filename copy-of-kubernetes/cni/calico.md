# Calico

## Cheatsheet

Get nodes information

```
calicoctl get nodes
```

Get IPAM information

```
calicoctl ipam show
```



## Installation / Configuration

{% embed url="https://projectcalico.docs.tigera.io/maintenance/clis/calicoctl/install" %}

### Microk8s

* Download calicoctr binary -> /usr/local/bin

After that it works w/o any additional config.&#x20;



Though you may experiment with two options

1\) Command line config:  `DATASTORE_TYPE=kubernetes KUBECONFIG=~/.kube/config ./calicoctl --allow-version-mismatch get nodes`

2\) Create config in /etc/calico/calicoctl.cfg:

```yaml
apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  datastoreType: "kubernetes"
  kubeconfig: "/home/ikuchin/.kube/config"
```

