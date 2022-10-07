# flux

## Cheatsheet

Version:

```
flux check
```

Flux entities:

* source - k8s manifests repository
* kustomize - k8s kustomize manifests
* image repository - image updater repositories
* image policy - image update policies
* image updater automation - single object for the whole cluster

Get entities

```
flux get [source|kustomize] -w
```

Get status of image updater per registry.&#x20;

```
flux get image repository

connme-images           2022-05-15T18:50:54Z    False           True    successful scan, found 12 tags
connme-cron             2022-05-15T18:50:54Z    False           True    successful scan, found 12 tags
connme-app              2022-05-15T18:50:54Z    False           True    successful scan, found 14 tags
connme-uploaders        2022-05-15T18:50:54Z    False           True    successful scan, found 2 tags 
connme-admin            2022-05-15T18:51:07Z    False           True    successful scan, found 1 tags 
connme-ui               2022-05-15T18:51:07Z    False           True    successful scan, found 2 tags 
```

Get image matching flux-policy

```
flux get image policy 

imagepolicy/connme-app     registry.conn-me.ru:5000/connme.ru-app:v1.1.19        True    Latest image tag for 'registry.conn-me.ru:5000/connme.ru-app' resolved to: v1.1.19      
imagepolicy/connme-images  registry.conn-me.ru:5000/connme.ru-images:v1.1.20     True    Latest image tag for 'registry.conn-me.ru:5000/connme.ru-images' resolved to: v1.1.20   
imagepolicy/connme-cron    registry.conn-me.ru:5000/connme.ru-cron:v1.1.20       True    Latest image tag for 'registry.conn-me.ru:5000/connme.ru-cron' resolved to: v1.1.20     
imagepolicy/connme-uploadersegistry.conn-me.ru:5000/connme.ru-uploaders:v1.1.20  True    Latest image tag for 'registry.conn-me.ru:5000/connme.ru-uploaders' resolved to: v1.1.20
imagepolicy/connme-admin   registry.conn-me.ru:5000/connme.ru-admin:v1.1.18      True    Latest image tag for 'registry.conn-me.ru:5000/connme.ru-admin' resolved to: v1.1.18    
imagepolicy/connme-ui      registry.conn-me.ru:5000/connme.ru-ui:v1.1.19         True    Latest image tag for 'registry.conn-me.ru:5000/connme.ru-ui' resolved to: v1.1.19       
```

Suspend entity from tracking&#x20;

```
flux suspend/resume [source|kustomize]
```



## Initial pre-check

```
flux check --pre
```

{% hint style="warning" %}
If initial pre-check fails with message "can't connect to API server".&#x20;

Flux can't find `.kube/config` file due client.config located in another place. Make a soft link like described [here](../kubernetes/microk8s.md#.kube-config).
{% endhint %}

## ERROR: image-updater -> early EOF

[Github issue](https://github.com/fluxcd/image-automation-controller/issues/315)&#x20;

TLDR

Github SSH deployment key must be read-write, this error points out that SSH key is read-only.

## flux autocompletion

Add below snippet to `~/.bashrc`

```
# flux completion
command -v flux >/dev/null && . <(flux completion bash)
```

## Local registry

Flux doesn't support insecure registry. Local registry must be configured with CA TLS. More on that topic [here](../docker/registry.md#registry-tls-encryption-with-ca).&#x20;

Flux configuration described [here](https://fluxcd.io/docs/cmd/flux\_create\_image\_repository/).

Basic steps:

Step 1) Create TLS-secret with rootCA certificate

```
flux create secret tls local-registry-cert --ca-file ./rootCA.pem
```

Step 2) Create image repository referencing root certificate

```
  flux create image repository app-repo \
    --cert-ref local-registry-cert \
    --image local-registry:5000/app --interval 5m
```

## Upgrade

1. [Download ](https://github.com/fluxcd/flux2/releases)flux-cli that you intend to upgrade to
2. Run bootstrap command. Below command intended for **microk8s dev-cluster**

```
flux bootstrap github \
  --components-extra=image-reflector-controller,image-automation-controller \
  --owner=ivankuchin \
  --repository=k8s-flux \
  --branch=main \
  --path=clusters/dev/microk8s \
  --read-write-key \
  --personal
```

3\. Wait or reconcile

More details are here:

{% embed url="https://fluxcd.io/docs/installation/#bootstrap-upgrade" %}

