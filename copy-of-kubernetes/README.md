# Copy of Kubernetes

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

## Cheatsheet

To get info about cluster config (for ex: control plane IP, etc…)

```
kubectl cluster-info
```

Service types

* ClusterIP - internal ip inside Cluster
* NodePort - expose single port on node

```
kubectl get all --all-namespaces
kubectl get (deployments|services|replicasets|pods) -o [wide|json|yaml]
kubectl get endpoints (list all service endpoints)
kubectl describe (deployment|service|pods) <name>
```

```
kubectl create deployment my_deployment --image nginx
kubectl expose deployment my_deployment --type=NodePort
```

(Once deployment exposed it could be reached via "internal service IP" or "external service IP" )

```
kubectl logs my_deployment-xxxxxxx-xxx
```

Forward single port to pod/deployment:

```
kubectl port-forward [deplyment/service/pod]/<name> node_port:k8s_port
```

K8s control plane component statuses

```
kubectl get componentstatuses
```

Collect api versions supported by cluster (incl: autoscaling, cert-manager, etc ....)

```
kubectl api-versions
```

Detailed description of API-object

```
kubectl explain cronjobs.metadata
```

If API allowed to be called by the user ?

```
kubectl auth can-i list pods --as=system:serviceaccount:default:mysvcaccount1
```

Run task as another user:

```
get pods -v=6 --as=system:serviceaccount:default
```

Increase verbosity level to observe API-calls

```
kubectl -v 6 exec app-deployment-6c9ccd5b94-5jc84 -- pwd
```

{% code lineNumbers="true" %}
```
loader.go:372] Config loaded from file:  C:\Users\ikuchin\.kube\config
round_trippers.go:553] GET https://172.16.17.69:16443/api/v1/namespaces/www-infomed-stat-ru/pods/app-deployment-6c9ccd5b94-5jc84 200 OK in 460 milliseconds
Defaulted container "app-container" out of: app-container, log-app-container, init-mysql (init)
podcmd.go:88] Defaulting container name to app-container
round_trippers.go:553] POST https://172.16.17.69:16443/api/v1/namespaces/www-infomed-stat-ru/pods/app-deployment-6c9ccd5b94-5jc84/exec?command=pwd&container=app-container&stderr=true&stdout=true 101 Switching Protocols in 478 milliseconds
```
{% endcode %}

Line (2) - 1-st API-call: check that pod exists

Line (3) - reply from API-server

Line (5) - 2-nd API-call: actual exec

## Debugging

{% embed url="https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/#ephemeral-container" %}
Debugging with an ephemeral container
{% endembed %}

Add another container to an existing pod. (Yes, this is correct !!! - attach container to a pod). Similar to `kubectl exec` but attaching whole container to a pod that is up and running.&#x20;

```
kubectl debug -it container_name --image=busybox --target=target_container --share-process
```

{% hint style="warning" %}
* This feature must be supported by CRI.
* \--target is an optional parameter pointing to container to attach to
* \--share-process allows the containers in this Pod to see processes from the other containers in the Pod
{% endhint %}

```
$ kubectl get pod
NAME   READY   STATUS    RESTARTS   AGE
web    2/2     Running   0          13s
$ kubectl debug web --image=alpine -it -- sh
/ #
```

{% code lineNumbers="true" %}
```
kubectl describe pod web
...
Containers:
  web:
    Image:          nigelpoulton/nginxadapter:1.0
    ...
  transformer:
    Image:         nginx/nginx-prometheus-exporter
    ...
Ephemeral Containers:
  debugger-j7lcq:
    Container ID:  docker://3abe8a589be26d1fe9200249c83931124b108d86063bbf967510c82005ed8967
    Image:         alpine
    Image ID:      docker-pullable://alpine@sha256:bc41182d7ef5ffc53a40b044e725193bc10142a1243f395ee852a8d9730fc2ad
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
    State:          Running
      Started:      Fri, 09 Sep 2022 20:56:11 -0400
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:         <none>
```
{% endcode %}

Lines (10) - onward: output information about debugging/ephemeral container.



## API&#x20;

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

Similar output can be retrieved via

```
kubectl explain cronjobs
```

{% code lineNumbers="true" %}
```
KIND:     CronJob
VERSION:  batch/v1

DESCRIPTION:
     CronJob represents the configuration of a single cron job.

FIELDS:
   apiVersion   <string>
```
{% endcode %}

Lines (1) - (2) shows kind and version.



## Proxy service

Dashboard/Proxy (similar to minikube or microk8s dashboard, it is entrypoint to K8s Cluster):

```
kubectl proxy --port=8080
kubectl get services <service name> -o yaml | grep -i self
  selfLink: /api/v1/namespaces/default/services/my-depl
curl http://localhost:8080/api/v1/namespaces/default/services/my-depl/          (gives back service config)
curl http://localhost:8080/api/v1/namespaces/default/services/my-depl/proxy/    (gives back content that service is assigned to)
curl http://localhost:8080/api/v1/namespaces/default/services/my-depl:80/proxy/ (gives back content that service is assigned to)
```

## MetalLB (L2 mode)

Select the IP-address pool dedicated to load-balancer

{% hint style="warning" %}
[documentation](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/#a-pure-software-solution-metallb) says it can’t be a host port, but it works
{% endhint %}

I tested 2 options – both works

a.       192.168.170.128 - 192.168.170.132 (Pool dedicated to LB)

b.       192.168.170.36 - 192.168.170.36 (Node IP)

```
microk8s.kubectl expose deployment my-depl --port 80 --target-port 80 --type LoadBalancer
```

```
microk8s.kubectl get services
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)        AGE
kubernetes   ClusterIP      10.152.183.1     <none>            443/TCP        2d18h
first        ClusterIP      10.152.183.45    <none>            80/TCP         2d18h
my-depl      LoadBalancer   10.152.183.133   192.168.170.128   80:32015/TCP   4h24m
```

Line (5) shows that LB `my-depl` been created

Then my-depl service available either over 192.168.170.128:80 or NodeIP:32015

```
curl 192.168.170.128 OR curl 192.168.170.36:32015
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>

```

{% hint style="warning" %}
LB do source NAT, so every client has SRC IP == Node IP
{% endhint %}

![](../.gitbook/assets/k8s\_metallb.png)

## Port-forwarding

```
$ microk8s.kubectl get all -n kube-system
NAME                                             READY   STATUS    RESTARTS   AGE
pod/metrics-server-8bbfb4bdb-k79ft               1/1     Running   10         2d18h
pod/kubernetes-dashboard-7ffd448895-n2l6j        1/1     Running   10         2d18h
pod/dashboard-metrics-scraper-6c4568dc68-nbc6r   1/1     Running   10         2d18h
pod/coredns-86f78bb79c-2sgnh                     1/1     Running   9          2d18h
pod/calico-node-lcjg8                            1/1     Running   13         2d18h
pod/calico-kube-controllers-847c8c99d-4trx2      1/1     Running   11         2d18h

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns                    ClusterIP   10.152.183.10    <none>        53/UDP,53/TCP,9153/TCP   2d18h
service/metrics-server              ClusterIP   10.152.183.82    <none>        443/TCP                  2d18h
service/kubernetes-dashboard        ClusterIP   10.152.183.219   <none>        443/TCP                  2d18h
service/dashboard-metrics-scraper   ClusterIP   10.152.183.213   <none>        8000/TCP                 2d18h

NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/calico-node   1         1         1       1            1           kubernetes.io/os=linux   2d18h

```

Line (13) shows that dashboard is active. Let's use dashboard to experiment with port-forwarding.

```
microk8s.kubectl port-forward service/kubernetes-dashboard 10443:443 --address=192.168.170.36 -n kube-system
```

Now service available via NodeIP:10443

```
$ curl 192.168.170.36:10443
Client sent an HTTP request to an HTTPS server.

$ curl https://192.168.170.36:10443
curl: (60) SSL certificate problem: self signed certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.

$ curl https://192.168.170.36:10443 -k
<!--
Copyright 2017 The Kubernetes Authors.

Licensed under the Apache License, Version 2.0 (the "License");

```

## Access to cluster from outside: minikube loadbalancer + tunneling

1. Create service type LoadBalancer&#x20;
2. Separate window `minikube tunnel` this will create static route via tunnel inside cluster&#x20;
3. Identify EXTERNAL IP from `kubectl get service`&#x20;
4. Test with curl \<EXTERNAL IP>

## Expose service on EXTERNAL IP

```
apiVersion: v1
kind: Service
metadata:
  name: ext-ip
  labels:
    app: ext-ip-service
spec:
  type: NodePort
  externalIPs: [10.22.0.0]
  selector:
    app: ext-ip-pod
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

```

Line (9) attaches `ExternalIP` to service

## Mount k8s variable to pod

```
  volumes:
  - name: kube-api-access-45dgd
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
```

Lines (14) - (19) shows how to mount var value to a pod&#x20;

## ISSUE in nginx (ERROR: 413 Request Entity Too Large)

Turn off request size by using 0 in annotation parameter, otherwise set max value of the payload (for ex: 50m)

```
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "0"
```

## Container start-up dependency

Detailed description: [https://dzone.com/articles/kubernetes-demystified-solving-service-dependencie](https://dzone.com/articles/kubernetes-demystified-solving-service-dependencie)

* Liveness probe: This probe is mainly used to determine if the container is in the Running state. For example, it can detect service deadlocks, slow responses, and other situations.&#x20;
* Readiness probe: This probe is mainly used to determine if the service is already working normally.&#x20;
* Readiness probes cannot be used in init containers.&#x20;
* If the pod restarts, all of its init containers must be run again.

Container that is dependent on another container must wait till it’s service name will be resolvable.

For example: if chat deployment depends on db, then in chat yaml

```
      initContainers:
      - name: init-mysql
        image: busybox
        command: ['sh', '-c', 'until nslookup db-service; do echo waiting for mysql; sleep 2; done;']
```

(\*) where db-service is service name associated with MySQL.

## Basic HTTPS encryption

Generate key and certificate

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key -out cert -subj "/CN=www.connme.microk8s.conn-me.ru/O=www.connme.microk8s.conn-me.ru"
```

Check certificate CN

```
openssl x509 -in cert -text -noout
```

Create k8s secret

```
kubectl create secret tls https-secret --key ./key --cert ./cert
```

Check k8s secret

```
get secrets https -o yaml
```

Add tls encryption to ingress object (file: ingress.yaml)

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: www-connme-ru
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "0"
spec:
  tls:
  - hosts:
    - www.connme.microk8s.conn-me.ru
    secretName: https
  rules:
  - host: www.connme.microk8s.conn-me.ru

```

TLS-encryption configured on lines (9) - (12)

## Let's Encrypt cert-manager

Github repository has link to cert-manager.io documentation. Unfortunately I was not able to find single paper describing whole process from start to finish.

{% embed url="https://github.com/jetstack/cert-manager/tree/v1.6.1" %}

{% hint style="info" %}
This paper may be used for reference, but use it with caution due to it is based on very old cert-manager version 0.16.1 and some objects has changed names: [https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes)
{% endhint %}

Install cluster wide cert-manger ([https://cert-manager.io/docs/installation/](https://cert-manager.io/docs/installation/))

```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.6.1/cert-manager.yaml
```

check installation

```
microk8s.kubectl get namespaces
microk8s.kubectl get pod -n cert-manager
```

After that install kubectl plugin `cert-manager` ([https://cert-manager.io/docs/usage/kubectl-plugin/](https://cert-manager.io/docs/usage/kubectl-plugin/)) it will give ability to use&#x20;

```
kubectl cert-manger
```

```
curl -L -o kubectl-cert-manager.tar.gz https://github.com/jetstack/cert-manager/releases/latest/download/kubectl-cert_manager-linux-amd64.tar.gz
tar xzf kubectl-cert-manager.tar.gz
sudo mv kubectl-cert_manager /usr/local/bin
```

Check cert-manager installation

```
kubectl cert-manager check api
```

Create _staging_ cluster issuer [https://cert-manager.io/docs/configuration/acme/#creating-a-basic-acme-issuer](https://cert-manager.io/docs/configuration/acme/#creating-a-basic-acme-issuer)

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: your_email_address_here
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: letsencrypt-staging
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          serviceType: ClusterIP
```

{% hint style="warning" %}
This is staging issuer, use it for testing. This issuer will enroll certificates that will have unknown RootCA.

Production issuer has rate limits: [https://letsencrypt.org/docs/rate-limits/](https://letsencrypt.org/docs/rate-limits/)&#x20;
{% endhint %}

Here is the production issuer:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: your_email_address_here
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          serviceType: ClusterIP
```

{% hint style="info" %}
Once ClusterIssuer created it is cluster-wide. No need to create it in every namespace.&#x20;
{% endhint %}

Check both cluster-issuers

```
kubectl get clusterissuer -n cert-manager
```

Expected output

```
NAME                  READY   AGE
letsencrypt-staging   True    5m16s
letsencrypt-prod      True    3m
```

Configure HTTPS on a required domain by simply adding tls section to ingress. Ingress object will generate self-signed certificate. Later it will be replaced with Let's Encrypt certificate.

Add annotation to ingress so it will understand that cluster-issuer must be involved into certifcate enrollment.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: www-connme-ru
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "0"
    cert-manager.io/cluster-issuer: "letsencrypt-staging"
spec:
  tls:
  - hosts:
    - www.connme.microk8s.conn-me.ru
    secretName: https-secret
  rules:
  - host: www.connme.microk8s.conn-me.ru
    http:
    ...
```

Line (8) - Instructs cert-manger to create a certificate, then cert-manager will update or create a ingress resource and use that to validate the domain. Once verified and issued, cert-manager will create or update the secret defined in the certificate.

Check status:

```
kubectl cert-manager status certificate https-secret

kubectl get order
kubectl get certificate
```

### Troubleshooting

Troubleshooting very well described here: [https://cert-manager.io/docs/faq/troubleshooting/](https://cert-manager.io/docs/faq/troubleshooting/)

During deployment same issue has happened. [https://github.com/jetstack/cert-manager/issues/4562](https://github.com/jetstack/cert-manager/issues/4562)&#x20;

TLDR; cert-manager created ingress that became inactive due to `class: nginx` yaml-config. This ingress has not been used in traffic service.

Logs from ingress controller shows app-service instead of cm-acme-http-xxxx:

10.1.128.216 - - \[03/Dec/2021:03:34:26 +0000] "GET /.well-known/acme-challenge/agGSsHvYvPJRHY4\_XHLuxzp8KnzUQn2T76ZE7c0IaAU HTTP/1.1" 404 275 "http://www.connme.ru/.well-known/acme-challenge/agGSsHvYvPJRHY4\_XHLuxzp8KnzUQn2T76ZE7c0IaAU" "cert-manager/v1.6.1 (clean)" 294 0.002 <mark style="background-color:yellow;">\[www-connme-ru-app-service-80]</mark> \[] 10.1.128.205:80 275 0.000 404 3c0f4c592afd2a177b36dff463377f59

## Registry authentication

Create secret

```
microk8s.kubectl create secret docker-registry kubernetes-production-docker-registry \
            --docker-server=ghcr.io \
            --docker-username=user_name \
            --docker-password=__PASSWORD__ \
            --docker-email=myname@domain.tld
```

Add secret to a deployment:

```docker
apiVersion: v1
kind: Pod
metadata:
  name: testpod
spec:
  containers:
    - name: testcontainer
      image: ghcr.io/ivankuchin/bestbounty.ru-cron:latest
  imagePullSecrets:
    - name: docker-registry
```

## Autoscale (horizontal)

Autoscale number of pods from 1 to 10 once pod getting closer to 95% of pod-s requested CPU.

{% hint style="warning" %}
Metrics-server must be enabled to be able to measure pod utilization.
{% endhint %}

```
kubectl autoscale pod_name  --cpu-percent=95 --min=1 --max=10
```

Check current metrics of HPA (HorizaontalPodAutoscaler) by using get or describe

```
kubectl get hpa/application-cpu -o wide
```

```
kubectl describe hpa/application-cpu -o wide
```

## CronJobs accessing POD

{% embed url="https://stackoverflow.com/questions/41192053/cron-jobs-in-kubernetes-connect-to-existing-pod-execute-script" %}

## Env var value from ...

#### ConfigMap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      env:
        # Define the environment variable
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: special-config
              # Specify the key associated with the value
              key: special.how
  restartPolicy: Neveryaml
```

#### Metadata

```yaml
          env:
            - name: NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
```
