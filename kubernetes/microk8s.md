# Microk8s

## Cheatsheet

```
microk8s start
microk8s status
microk8s stop

microk8s enable <addon1> <addon2> ... 
microk8s enable dns dashboard metrics-server

Dashboard/proxy:
microk8s dashboard-proxy

microk8s config
```

## Microk8s image management

```
microk8s ctr image ls | grep registry.local
```

Remove image

```
microk8s ctr image rm xxxxxxxxxxxxxx
```

## Registry <a href="#microk8s_insecure_registry" id="microk8s_insecure_registry"></a>

### Insecure registry <a href="#microk8s_insecure_registry" id="microk8s_insecure_registry"></a>

{% embed url="https://microk8s.io/docs/registry-private" %}

Edit /var/snap/microk8s/current/args/containerd-template.toml and add

```
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."10.141.241.175:32000"]
          endpoint = ["http://10.141.241.175:32000"]
```

```
microk8s stop
microk8s start
```

### Local registry with local CA

Copy rootCA.pem to /etc/ssl/certs

```
microk8s stop
microk8s start
```

{% hint style="info" %}
No need to add anything to /var/snap/microk8s/current/args/container-template.toml
{% endhint %}

## Port forwarding on ingress nginx

{% embed url="https://microk8s.io/docs/addon-ingress" %}

{% embed url="https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services" %}

Step 1) if ConfigMap tcp-ingress-nginx has additional ports configured

```
microk8s.kubectl get configmaps -n ingress nginx-ingress-tcp-microk8s-conf -o yaml
```

Step 2) Patch ConfigMap tcp-ingress-nginx

```
microk8s.kubectl patch -n ingress --patch '{"data":{"7681":"www-connme-ru/chat-service:7681"}}' configmap/nginx-ingress-tcp-microk8s-conf
```

Step 3) check again

```
microk8s.kubectl get configmaps -n ingress nginx-ingress-tcp-microk8s-conf -o yaml
```

Make sure that you see port 7681

```
apiVersion: v1
data:
  "7681": www-connme-ru/chat-service:7681
```

Step 4) get ports exposed by ingress service

```
microk8s.kubectl get daemonsets -n ingress nginx-ingress-microk8s-controller -o jsonpath='{.spec.template.spec.containers[0].ports}' | jq
```

Expected output:

```
[
  {
    "containerPort": 80,
    "hostPort": 80,
    "name": "http",
    "protocol": "TCP"
  },
  {
    "containerPort": 443,
    "hostPort": 443,
    "name": "https",
    "protocol": "TCP"
  },
  {
    "containerPort": 10254,
    "hostPort": 10254,
    "name": "health",
    "protocol": "TCP"
  }
]

```

Step 5) inspect patch file

```
more ingress-nginx-controller-patch.yaml 
```

```
spec:
  template:
    spec:
      - name: nginx-ingress-microk8s
      containers:
        ports:
         - containerPort: 7681
           hostPort: 7681
```

Line(8)-(9) pay attention to port in those lines

Step 6) patch ingress service

```
microk8s.kubectl patch daemonsets -n ingress nginx-ingress-microk8s-controller --patch "$(cat ingress-nginx-controller-patch.yaml)"
```

Step 7) check ports exposed by ingress service

```
microk8s.kubectl get daemonsets -n ingress nginx-ingress-microk8s-controller -o jsonpath='{.spec.template.spec.containers[0].ports}' | jq
```

```
[
  {
    "containerPort": 7681,
    "hostPort": 7681,
    "protocol": "TCP"
  },
  {
    "containerPort": 80,
```

Lines (3) - (5) shows that service been successfully patched

## multipathd\[xxx]: sda: add missing path

{% embed url="https://askubuntu.com/questions/1242731/ubuntu-20-04-multipath-configuration" %}

Syslog flooded with

```
multipathd[651]: sda: add missing path
multipathd[651]: sda: failed to get udev uid: Invalid argument
multipathd[651]: sda: failed to get sysfs uid: Invalid argument
multipathd[651]: sda: failed to get sgio uid: No such file or directory
multipathd[651]: sda: add missing path
multipathd[651]: sda: failed to get udev uid: Invalid argument
multipathd[651]: sda: failed to get sysfs uid: Invalid argument
multipathd[651]: sda: failed to get sgio uid: No such file or directory
```

**Root cause:** VMWare by default doesn't provide information needed by udev to generate /dev/disk/by-id entries. Apart from ESX, VMWare Workstation (my case) is also affected. The resolution is to put to VM definition:

```
disk.EnableUUID = "TRUE"
```

![](../.gitbook/assets/k8s\_microk8s\_issue1\_1.jpg) ![](../.gitbook/assets/k8s\_microk8s\_issue1\_2.jpg) ![](../.gitbook/assets/k8s\_microk8s\_issue1\_3.jpg)

## .kube/config

config on microk8s buried deep in snap. If short path is required:

```
ln -s /var/snap/microk8s/current/credentials/client.config ~/.kube/config
```

## Microk8s version

```
snap info microk8s | grep tracking
```

```
microk8s.kubectl get nodes
```

## Token

API token might be taken from [client.config](microk8s.md#.kube-config)

## Ingress SSL pass-through

Some microservices expose HTTPS instead of HTTP, client must pass-through HTTPS-connection all the way down to microservice instead of terminating proxying it through ingress. For example: ArgoCD ([https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#kubernetesingress-nginx](https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#kubernetesingress-nginx))

There are two ways to tackle the issue:&#x20;

#### Option 1: SSL pass-through:

Let client pass ingress through. There will be alone session from client to microservice.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
spec:
  rules:
  - host: argocd.microk8s.conn-me.ru
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 443
```

Lines (6) - (7) forces SSL to pass-through

{% hint style="warning" %}
Caveat that _MAIN_ ingress must be run with --enable-ssl-passthrough
{% endhint %}

Edit DaemonSet of ingress controller to add parameter

```
microk8s.kubectl edit ds -n ingress nginx-ingress-microk8s-controller
```

```
spec:
  containers:
  - args:
    - /nginx-ingress-controller
    - --configmap=$(POD_NAMESPACE)/nginx-load-balancer-microk8s-conf
    - --tcp-services-configmap=$(POD_NAMESPACE)/nginx-ingress-tcp-microk8s-conf
    - --udp-services-configmap=$(POD_NAMESPACE)/nginx-ingress-udp-microk8s-conf
    - --enable-ssl-passthrough
    - --ingress-class=public
    - ' '
    - --publish-status-address=127.0.0.1
```

Line (8) added

#### Option 2: Make backend protocol HTTPS (preferred)

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  rules:
  - host: argocd.microk8s.conn-me.ru
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 443
```

Line (6) added

In this case it will work as regular ingress but backend protocol will be HTTPS.
