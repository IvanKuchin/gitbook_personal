# Docker

## Cheatsheet

Create customized image from container (having the ability to commit is docker’s way of taking the resulting state from that process and preserving it.)

```
docker commit running_container new_image_name
```

Clean up docker system (incl: cached image layers, stopped containers, unused volumes, unused networks)

```
docker system prune --all
```

Image layers size

```
docker history <image_id>
```

## Run container as a non-root-user inside

```
FROM httpd:2.4

RUN apt -y update
RUN apt -y install netcat libcap2-bin procps 

ENV LC_ALL en_US.UTF-8
ENV LANG=en_US.UTF-8

RUN setcap "cap_net_bind_service=+ep" /usr/local/apache2/bin/httpd
RUN getcap /usr/local/apache2/bin/httpd

RUN chown -R www-data:www-data /usr/local/apache2

HEALTHCHECK --interval=60s --timeout=30s CMD nc -zv http://localhost:80 || exit 1

USER www-data

EXPOSE 80

CMD ["/usr/local/apache2/bin/httpd", "-D", "FOREGROUND"]
```

Line (14) HEALTHCHECK is interesting because I haven’t used it before.

Line (16) containerized user is www-data

Lines (9) and (10), due to containerzed user is www(non-priv), he won't be able to run www-server on port 80. Ports below 1024 require root privileges. setcap – allows non-privilege USER to open socket below 1024 range.

{% hint style="warning" %}
* Apache config /usr/local/apache2/conf/httpd.conf has explicit user and group daemon. Those settings might be different from running environment.\
  \*) running env will run apache from USER www-data\
  \*) but config still have another user daemon. This might add a lot of confusion during troubleshooting.
* There won't be access to root-shell therefore, no way to use apt. This is significant hurdle during troubleshooting.
{% endhint %}

## Manifest for supported container platforms

```
docker run --rm mplatform/mquery crazymax/swarm-cronjob:latest
```

Output shows supported platforms

```
Image: crazymax/swarm-cronjob:latest
 * Manifest List: Yes
 * Supported platforms:
   - linux/amd64
   - linux/arm/v6
   - linux/arm/v7
   - linux/arm64
   - linux/386
   - linux/ppc64le
   - linux/s390x
```

## Win10: Docker desktop + WSL path to volume files

&#x20;c:\Users\ikuchin\AppData\Local\Docker\wsl\data\ext4.vhdx

## BuildKit

To enable BuildKit set the env variable

```
set DOCKER_BUILDKIT=1
```

then

```
docker build -t ikuchin/connme:test -f Dockerfile_to_build .
```

## How to reduce container size

To reduce size of final container I wanted to build all binaries using static linking and simple copy them over to base-container. But turns out that many libraries don’t provide static libraries by default. For example:

* ImageMagick, doesn’t include static version, you can build _\_\_\_\_.a_ from source and then add all libraries: jpeg, tiff, gif, bmp, png, scale, openmp, etc…
* WkhtmlToX comes with shared library only

Final decision is to live with what docker can build and later switch over to buildah, once GitHub actions begin to support it.

## Non-interactive installation

Set environment variable:

```
export DEBIAN_FRONTEND noninteractive
export DEBCONF_NONINTERACTIVE_SEEN true
```

#### Mysql example

To install mysql w/o interactive entering root credentials. Debconf must be used

```
root@a130aedbeea9:/backend# debconf-show mysql-server
root@a130aedbeea9:/backend# debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
root@a130aedbeea9:/backend# debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'
root@a130aedbeea9:/backend# debconf-show mysql-server
* mysql-server/root_password_again: (password omitted)
* mysql-server/root_password: (password omitted)
```

#### tzdata example in docker

```
FROM ubuntu:focal as base

############################
# make apt non-interactive #
############################
ENV DEBIAN_FRONTEND noninteractive
ENV DEBCONF_NONINTERACTIVE_SEEN true

####################
# preseed timezone #
####################
RUN echo tzdata tzdata/Areas select Europe > /tmp/preseed.txt \
 && echo tzdata tzdata/Zones/Europe select Moscow >> /tmp/preseed.txt \
 && debconf-set-selections /tmp/preseed.txt
 
RUN debconf-show tzdata
```



## Mail delivery from a container

Container allows single daemon to run, means that sendmail daemon is not an option in containerized environment. Use SSMTP micro-tool to workaround limitation.

{% embed url="https://www.bonusbits.com/wiki/HowTo:Configure_SendMail_to_Use_SMTP_Relay" %}

## Redirect log files to stdout & stderr

```
RUN ln -sf /dev/stdout /home/httpd/www/logs/access.log \
 && ln -sf /dev/stderr /home/httpd/www/logs/error.log
```

\-f - will remove destination file (access.log and error.log) making them linked to stdout & stderr.

## Alpine vs scratch

Alpine has `/bin/sh` which helps to troubleshoot basic issues.&#x20;

Scratch has no means to get inside container.&#x20;

Final container sizes:

* Example binary size - 6MB
* Scratch based image: size is 16MB and 1 RootFS layer
* Alpine based image: size is 18MB and 3 RootFS layers

Scratch-based software BoM:

```
C:\Users\ikuchin>docker sbom test_scratch
NAME                    VERSION  TYPE
example.com/gorilla              go-module
github.com/gorilla/mux  v1.8.0   go-module
```

Alpine-based software BoM:

```
C:\Users\ikuchin>docker sbom test_alpine
NAME                    VERSION      TYPE
alpine-baselayout       3.2.0-r18    apk
alpine-keys             2.4-r1       apk
apk-tools               2.12.7-r3    apk
busybox                 1.34.1-r3    apk
ca-certificates-bundle  20191127-r7  apk
example.com/gorilla                  go-module
github.com/gorilla/mux  v1.8.0       go-module
libc-utils              0.7.2-r3     apk
libcrypto1.1            1.1.1l-r7    apk
libretls                3.3.4-r2     apk
libssl1.1               1.1.1l-r7    apk
musl                    1.2.2-r7     apk
musl-utils              1.2.2-r7     apk
scanelf                 1.3.3-r0     apk
ssl_client              1.34.1-r3    apk
zlib                    1.2.11-r3    apk
```

## Look inside docker-image

```
dive <docker image id>
```

{% embed url="https://github.com/wagoodman/dive" %}

## Bash scripting

Inside `Dockerfile` bash parameter expansion can be used with expressions similar to&#x20;

> RUN git clone --depth 1 https://github.com/IvanKuchin/<mark style="background-color:yellow;">${BUILD\_REPO%%-\*}</mark>.git /tmp/${BUILD\_REPO}/

```bash
REPO=infomed-stat.ru-admin

# remove long prefix
echo ${REPO%%-*}
infomed

# remove prefix
echo ${REPO%-*}
infomed-stat.ru
```

{% embed url="https://devhints.io/bash" %}
