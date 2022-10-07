# Server customization

## vsftpd

Configuration of /etc/vsftpd.conf

```
lock_upload_files=NO
write_enable=YES
chroot_local_users=YES
allow_writeable_chroot=YES
```

Без lock\_upload\_files окончание загрузки фаила на сервер «замирает» примерно на минуту. Если-же отключиь file\_lock – такого не происходит.

Sublime integration with cgi-bin, html paths require additional user (ftp\_connme)

```
mkdir /var/ftp/ftp_connme
mount --bind /home/httpd/www.connme.ru/ /var/ftp/ftp_connme/
usermod -d /var/ftp/ftp_connme/ ftp_connme
```

To mount bindfs at a boot time add below line to /etc/fstab

```
/home/httpd/www.connme.ru       /var/ftp/ftp_connme     none    defaults,bind   0       0
```

## GoAccess apache log analyzer

{% embed url="https://goaccess.io" %}

Container reads apachelogs and produces output.html, container doesn't have built-in web-server to watch logs you must run web-server in side-container. If real-time log analysis required, then container expose WebSocket port 7890 which generates real-time logs for client to consume.

## Dynamic DNS

Client sending “update”-message to server. Server allowing update zone by IP-range or by key. Generate key-pair on client

```
dnssec-keygen -a HMAC-MD5 -b 512 -n HOST <keyname>
```

Server configuration (BIND9):

```
key "dev." {
    algorithm hmac-md5;
    secret "BYlaKYtBHl…..Lhpk9YUQ==";
};

zone "dynamic.conn-me.ru" {
    type master;
    allow-update { key "dev."; };
};
```

Client configuration:

```
#!/bin/bash


# getting the public IP
ip=`curl ipinfo.io/ip 2>/dev/null`

# file with private key
KEY="/home/ikuchin/ddns_update/auth_key"

# zone update
cat <<EOF | nsupdate -k $KEY
server 185.68.152.8
zone dynamic.conn-me.ru
update delete backup.dynamic.conn-me.ru. A
update add backup.dynamic.conn-me.ru. 60 A $ip
show
send
EOF
```

Auth-key features on a client:

* name **must** be the same as on the server
* key should be public (with _space_), not a private key

```
key "dev." {
    algorithm hmac-md5;
    secret "6fAJEUc+asppo…. ………+CwpikNWdQ==";
};
```

## «Зависшие» соединения

```
netstat –ntpec --timer
```

Flags:&#x20;

\-c – continuos update&#x20;

\-n – numeric output, no DNS resolution&#x20;

\-t – Internet connections only, no Unix sockets&#x20;

\-p – add PID information&#x20;

\-e – extra info (includes User and Inode)&#x20;

\--timer – includes timer information described above.



Output example:

> keepalive (6176.47/0/0)&#x20;
>
> <1st field> <2nd field>

The 1st-field can have values:\
keepalive - when the keepalive timer is ON for the socket\
on - when the retransmission timer is ON for the socket\
off - none of the above is ON

The 2nd-field has THREE subfields:\
(6176.47/0/0) -> (a/b/c)\
a=timer value (a=keepalive timer, when 1st field="keepalive"; a=retransmission timer, when 1st field="on")\
b=number of retransmissions that have occurred\
c=number of keepalive probes that have been sent

When the client went down, server starts to send _keepalive_ messages (c-subfield of 2’nd-field will be incrementing) to check connectivity to client.\
If server tried to send data to client at that time, server will not receive TCP ACK from client, server starts to _retransmit_ data __ (b-subfield of 2’nd-field will be incrementing).

## NFS CIFS mounts

Add fllowing to /etc/fstab

> //192.168.168.32/ikuchin /storage/ikuchin <mark style="background-color:yellow;">cifs</mark> credentials=/home/ikuchin/.smbcredentials,uid=ikuchin,gid=ikuchin 0 0
>
> 192.168.168.32:/nfs/ikuchin /storage/ikuchin <mark style="background-color:yellow;">nfs</mark> defaults 0 0

If samba requires authentication, add credentials to a file /home/ikuchin/.smbcredentials

```
username=admin
password=__PASSWORD__
domain=
```

## cron.daily test

```
sudo run-parts -v /etc/cron.daily
```

## HTTPS enablement

Key pair and CSR generation

```
openssl genrsa -out connme.ru.key 2048
openssl req -new -sha256 -key connme.ru.key -out connme.ru.csr
```

or self-signed

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout mysitename.key -out mysitename.crt
```

Apache configuration:

Enable Virtual naming in HTTPS

```
NameVirtualHost *:443
```

Virtual host configuration

```
SSLEngine on
SSLProtocol all -SSLv2 -SSLv3
SSLCertificateFile xxx.crt
SSLCertificateKeyFile xxx.key
SSLCertificateChainFile xxx.ca-bundle
```

## Сlear terminal window

```
printf “\033c”
```

## Time zone

Проверка текущего часового пояса.

```
timedatectl
```

[Список всех часовых поясов](https://en.wikipedia.org/wiki/List\_of\_tz\_database\_time\_zones)

```
timedatectl list-timezones
```

or

```
ls /usr/share/zoneinfo/
```

Смена часового пояса

```
sudo timedatectl set-timezone EST
```

or

```
ln –sf /usr/share/zoneinfo/America/New_York /etc/localtime
```

## Process thread information

```
pstree –p
ps –T –p <PID>
top –H –p <PID>
```

## Fedora FW

```
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-port=2049/tcp
firewall-cmd --permanent --add-port=2049/udp
firewall-cmd –-reload
```

## Apache image type treatment



Во избежание «взлома сервера» путем загрузки скрипта в директорию /images.

Сервер проверяет формат загружаемых фаилов

* Если картинка – загрузка разрешается
* Если НЕ картинка – временный фаил удаляется из папки /tmp

Apache2 будет проверять формат отдаваемого фаила:

* Если картинка \\.(gif|jpe?g|png|ttf)$ - «разрешить показ»
* Если что-либо другое отдавать как обычный фаил

```
ForceType application/octet-stream
Header set Content-Disposition attachment
Header set X-Content-Type-Options nosniff
<FilesMatch "(?i)\.(gif|jpe?g|png|ttf)$">
    ForceType none
    Header unset Content-Disposition
</FilesMatch>
```

Headers module must be enabled

```
sudo a2enmod headers
sudo service apache2 restart
```

## Apache referrer check with .htaccess

First read this article: [http://tltech.com/info/referrer-htaccess/](http://tltech.com/info/referrer-htaccess/)

Then read below.

```
RewriteCond %{HTTP_HOST}@@%{HTTP_REFERER} !^([^.]+\.)?([^.]+\.[^@]+)@@https?:\/\/([^.]+\.)?\2\/.*
```

Assume htaccess will match against: [www.connme.ru@@https://images.connme.ru/](http://www.connme.ru%40@https/images.connme.ru/)

Complexity of that regexp is 28 cycles. Check it on [https://regex101](https://regex101)

![](../.gitbook/assets/apache\_regexp\_1.png)

{% hint style="info" %}
Specific to that match: it will match sub-string that does NOT contains “dot”. Which means that it will match only one sub-domain. If more than one required, replace it to (.+\\.)?, but it will increase complexity of regexp 3 times.
{% endhint %}



Complexity of matching multiple sub-domains is 99

![](../.gitbook/assets/apache\_regexp\_2.png)

## Apache cache control

Enable apache2 mod expire

```
sudo a2enmod expire
```

HTML pages expires in 30 secs

JS expires in 24 to speed-up loading. If JS must be changed:

1. change pacing to 10 mins ,
2. wait 24 hours
3. update scripts
4. change pacing to 24 hours

```
<VirtualHost *:80>
        <IfModule mod_expires.c>
                ExpiresActive on
                ExpiresDefault "access plus 30 seconds"
                ExpiresByType text/html "access plus 30 seconds"
                ExpiresByType text/js "access plus 24 hours"
                ExpiresByType text/javascript "access plus 24 hours"
                ExpiresByType image/gif "access plus 3 months"
                ExpiresByType image/jpg "access plus 3 months"
                ExpiresByType image/jpeg "access plus 3 months"
                ExpiresByType image/png "access plus 3 months"
                ExpiresByType video/webm "access plus 3 months"
                ExpiresByType video/mp3 "access plus 3 months"
        </IfModule>
</VirtualHost>
```

Specifics can be configured in .htaccess. The disadvantage of this method is that you **must** specify exact time of expiration.

```
<FilesMatch "logo_site.png">
    Header set Expires "Tue, 16 Jun 2020 20:00:00 GMT"
</FilesMatch>
```

Other way by using a relative timestamp is to configure .htaccess in a following way. This will affect all files in the directory.

```
ExpiresActive On
ExpiresDefault "access plus 1 day"
```

## SNMP config

#### Server

On a server in /etc/snmp/snmpd.conf comment restriction on systemOnly MIB

```
rocommunity  public default # -V systemonly
```

Reduce noise from snmpd in  /var/log/syslog ([https://serverfault.com/questions/310640/reduce-snmpd-logging-verbosity](https://serverfault.com/questions/310640/reduce-snmpd-logging-verbosity))

In /etc/default/snmpd replace -Lsd to -LSwd

```
SNMPDOPTS='-LSwd -Lf /dev/null -u snmp -g snmp -I -smux,mteTrigger,mteTriggerConf -p /run/snmpd.pid'
```

#### Client

On a client if all MIBs required:

Check what is available now:

```
snmptranslate -Tp
```

If output "almost" empty, then

```
sudo apt-get install snmp-mibs-downloader
sudo download-mibs
sudo sed -i "s/^\(mibs *:\).*/#\1/" /etc/snmp/snmp.conf
sudo service snmpd restart
```

Check again

```
snmptranslate -Tp
```

## Slow SSH-login due to dbus systemd-logind issues

```
journalctl -xe
```

If find messages related to org.freedesktop.login1 then&#x20;

{% embed url="https://bugzilla.redhat.com/show_bug.cgi?id=1532105" %}

Option 1) Manually restart `systemd-login`

```
busctl | grep org.freedesktop.login1
ps aux | grep systemd-logind -m 1
systemctl restart systemd-logind
busctl | grep org.freedesktop.login1
```

Option 2) restarting systemd-logind if for some reason dbus restarts

```
# mkdir -p /etc/systemd/system/systemd-logind.service.d

# cat > /etc/systemd/system/systemd-logind.service.d/bz1654779.conf << EOF
[Unit]
After=dbus.service
BindsTo=dbus.service
EOF

# systemctl daemon-reload
```
