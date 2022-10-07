# Registry

## Cheatsheet

```
curl http://localhost:5000/v2/_catalog
curl http://localhost:5000/v2/XXXXXX/tags/list
```

Get an image digest:

```
curl -v -s -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -X GET http://registry.local:5000/v2/connme.ru_app/manifests/v0.0.1 | grep Docker-Content-Digest
```

Delete the image from registry:

```
curl -v -s -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -X DELETE http://registry.local:5000/v2/connme.ru_app/manifests/<digest>
```

List of all actions available in registry:

{% embed url="https://github.com/distribution/distribution/blob/main/docs/spec/api.md#detail" %}

## Registry with self-signed certificate

{% embed url="https://docs.docker.com/registry/insecure#use-self-signed-certificates" %}

## Insecure registry

#### Buildah:

```
buildah pull --tls-verify=false localhost:5000/ikuchin/taskmaster
buildah push --tls-verify=false localhost:5000/ikuchin/taskmaster
```

#### Windows:

![](../.gitbook/assets/docker\_win\_insecure\_registry.png)

#### Microk8s insecure registry

Check in [here](../kubernetes/microk8s.md#microk8s\_insecure\_registry)

#### Minikube insecure registry

Check in [here](../kubernetes/minikube.md#insecure-registry)

## Registry TLS encryption with CA&#x20;

Best option is to setup Certificate Authority (CA) and rollout certificate specific to registry. Steps are taken from [here](https://unix.stackexchange.com/questions/451207/how-to-trust-self-signed-certificate-in-curl-command-line).

#### Step 1) generate root CA key / cert

```
openssl req -x509 -sha256 -days 1825 -newkey rsa:2048 -keyout rootCA.key -out rootCA.crt
```

check that generated certificate allowed to sign certificates.

```
openssl x509 -in rootCA.crt  -text -noout
```

Look for Basic constraint CA:TRUE

```
            X509v3 Basic Constraints: 
                CA:TRUE
```

#### Step 2) Generate registry key and certificate sign request (csr)

```
openssl genrsa -out registry.key 2048
openssl req -key registry.key -new -out registry.csr
```

#### Step3) Sign registry-csr with root CA key

{% hint style="warning" %}
Add extension file and add FQDN to SAN (X509v3 Subject Alternative Name).&#x20;

This is important for future TLS-checks.
{% endhint %}

```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
subjectAltName = @alt_names
[alt_names]
DNS.1 = registry.conn-me.ru
```

Sign registry.csr

```
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in registry.csr -out registry.crt -days 365 -CAcreateserial -extfile registry.ext
```

#### Step 4) (Optional) convert .crt -> .pem

Sometime .crt may include private key, therefore recommended way is to distribute .pem (which is .crt without private key).

```
openssl x509 -in rootCA.crt -out rootCA.pem
openssl x509 -in registry.crt -out registry.pem
```

#### Step 5) configure container with registry.key and registry.crt

```
version: "3"

# More info at https://docs.docker.com/registry/deploying/#deploy-your-registry-using-a-compose-file
services:
  registry:
    image: registry:2
    ports:
      - 5000:5000
    environment:
      REGISTRY_HTTP_TLS_CERTIFICATE: /certs/registry.crt
      REGISTRY_HTTP_TLS_KEY: /certs/registry.key
      # REGISTRY_AUTH: htpasswd
      # REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
      # REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
    volumes:
      - /storage/ikuchin/docker_registry:/var/lib/registry
      - ./certs:/certs
      - ./auth:/auth

    restart: unless-stopped
```

Lines (10) and (11) configure key and certificate

#### Step 6) test access to registry

Attempt that should fail due to unknown rootCA

```
curl https://registry.conn-me.ru:5000/v2/_catalog
```

```
curl: (60) server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none
More details here: http://curl.haxx.se/docs/sslcerts.htm
```

Successful attempt

```
curl --cacert rootCA.pem  https://registry.conn-me.ru:5000/v2/_catalog
```
