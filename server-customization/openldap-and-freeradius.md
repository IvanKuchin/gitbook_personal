# OpenLDAP and FreeRADIUS

## OpenLDAP installation

Follow steps from installation guide here: [https://scytalelabs.com/setup-and-configure-openldap-using-docker-image-on-ubuntu-16-04/](https://scytalelabs.com/setup-and-configure-openldap-using-docker-image-on-ubuntu-16-04/)

Or use docker-compose to run both: OpenLDAP and phpLDAPadmin

Content of docker-compose.yaml

```
version: "3"

services:
  ldap:
    container_name: ldap-service
    image: osixia/openldap:latest
    ports:
      - "389:389/tcp"
    environment:
      TZ: 'America/New_York'
      LDAP_ORGANISATION: 'conn-me'
      LDAP_DOMAIN: 'conn-me.ru'
      LDAP_ADMIN_PASSWORD: 'sdlkashhs'
      LDAP_BASE_DN: 'dc=conn-me,dc=ru'
    # Volumes store your data between container upgrades
    volumes:
      - './openldap/var/lib/ldap:/var/lib/ldap'
      - './openldap/etc/ldap/slap.d:/etc/ldap/slapd.d'
      - './openldap/etc/ldap/schema:/etc/ldap/schema'

    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"


  phpldapadmin:
    container_name: phpldapadmin-service
    hostname: phpldapadmin.conn-me.ru
    image: osixia/phpldapadmin:latest
    ports:
      - "8444:443/tcp"
    environment:
      TZ: 'America/New_York'
      PHPLDAPADMIN_LDAP_HOSTS: 'ldap-service'
    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
```

then

```
sudo docker-compose up --detach
```

if want to check what is happening on the console omit `--detach` flag.

{% hint style="warning" %}
Avoid using Ctrl+C to stop while inside container, otherwise containers in switch to a stop mode. Instead, in another terminal use sudo docker-compose down
{% endhint %}

## OpenLDAP client configuration

```
sudo apt install ldap-auth-config nscd
```

parameters to enter during installation

> base: dc=conn-me,dc=ru
>
> user: cn=admin,dc=conn-me,dc=ru

if package needs to be reconfigured:

```
sudo dpkg-reconfigure ldap-auth-config
```

Next step is to modify /etc/nsswitch.conf

```
passwd:         ldap files systemd
group:          ldap files systemd
shadow:         ldap files
```

Lines (1) - (3): add `ldap`keyword

Modify /usr/share/pam-configs/mkhomedir like shown below

```
Name: Create home directory on login
Default: yes
Priority: 900
Session-Type: Additional
Session-Interactive-Only: yes
Session:
        required                        pam_mkhomedir.so umask=022 skel=/etc/skel
```

{% hint style="warning" %}
Do NOT do pam-auth-update. It will revert above changes to a default.
{% endhint %}

Finally restart and enjoy

```
service nscd restart
```

#### Fallback from LDAP to local auth

Check out this link to get detailed explanation: [https://www.howtoforge.com/linux\_ldap\_authentication#client-configuration](https://www.howtoforge.com/linux\_ldap\_authentication#client-configuration)

By default LDAP will attempting authenticate user without falling back, even if LDAP-server went down. To fix that in `/etc/ldap.conf` change following params:

```
# Search timelimit
timelimit 5

# Bind/connect timelimit
bind_timelimit 5

# Reconnect policy: hard (default) will retry connecting to
# the software with exponential backoff, soft will fail
# immediately.
bind_policy soft
```

After that change server should be rebooted, probably not necessarily, but I'm yet to find the way to apply this change.

Authentication logs are in `/var/log/auth.log`

{% code lineNumbers="true" %}
```
Oct  2 03:06:29 ns1 sshd[3670]: pam_ldap: ldap_simple_bind Can't contact LDAP server
Oct  2 03:06:29 ns1 sshd[3670]: pam_ldap: reconnecting to LDAP server...
Oct  2 03:06:34 ns1 sshd[3670]: pam_ldap: ldap_simple_bind Can't contact LDAP server
Oct  2 03:06:37 ns1 sshd[3670]: Failed password for ikuchin from 47.14.39.244 port 57875 ssh2
```
{% endcode %}

Lines (1) - (3) entering LDAP pass with server unavailable

Line (4) 5 seconds after binding connection failed

## OpenLDAP troubleshooting

Run side container where install ldap-utils

```
  debug:
    container_name: debug
    image: freeradius/freeradius-server:latest
    restart: unless-stopped
    command: /bin/bash -c "sleep 3600"
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"

```

Run debug container connected to ldap-service network

```
sudo docker exec -it debug /bin/bash
apt update; apt install ldap-utils
```

Open another console with logs from ldap-service

```
sudo docker attach ldap-service
```

In debug container run below commands, watch for logs in ldap-service



#### Set SRCH filter: -s base '(uid=ikuchin)'

```
ldapsearch -x -b '' -s base '(uid=ikuchin)' namingContexts -D "cn=admin,dc=conn-me,dc=ru" -W -h ldap-service       
```

```
618fe870 conn=2778 fd=18 ACCEPT from IP=192.168.48.2:34994 (IP=0.0.0.0:389)
618fe870 conn=2778 op=0 BIND dn="cn=admin,dc=conn-me,dc=ru" method=128
618fe870 conn=2778 op=0 BIND dn="cn=admin,dc=conn-me,dc=ru" mech=SIMPLE ssf=0
618fe870 conn=2778 op=0 RESULT tag=97 err=0 text=
618fe870 conn=2778 op=1 SRCH base="" scope=0 deref=0 filter="(uid=ikuchin)"
618fe870 conn=2778 op=1 SRCH attr=namingContexts
618fe870 conn=2778 op=1 SEARCH RESULT tag=101 err=0 nentries=0 text=
618fe870 conn=2778 op=2 UNBIND
618fe870 conn=2778 fd=18 closed
```

Line (5): filter="....."

#### Set SRCH BASE: -b 'dc=conn-me,dc=ru'

```
ldapsearch -x -b 'dc=conn-me,dc=ru' -s base '(uid=ikuchin)' namingContexts -D "cn=admin,dc=conn-me,dc=ru" -W -h ldap-service
```

```
618fe8fe conn=2782 fd=18 ACCEPT from IP=192.168.48.2:35020 (IP=0.0.0.0:389)
618fe8fe conn=2782 op=0 BIND dn="cn=admin,dc=conn-me,dc=ru" method=128
618fe8fe conn=2782 op=0 BIND dn="cn=admin,dc=conn-me,dc=ru" mech=SIMPLE ssf=0
618fe8fe conn=2782 op=0 RESULT tag=97 err=0 text=
618fe8fe conn=2782 op=1 SRCH base="dc=conn-me,dc=ru" scope=0 deref=0 filter="(uid=ikuchin)"
618fe8fe conn=2782 op=1 SRCH attr=namingContexts
618fe8fe conn=2782 op=1 SEARCH RESULT tag=101 err=0 nentries=0 text=
618fe8fe conn=2782 op=2 UNBIND
618fe8fe conn=2782 fd=18 closed
```

Line (5): SRCH base="dc=conn-me,dc=ru"

#### Set SRCH BASE in OU user: -b 'ou=users,dc=conn-me,dc=ru'

```
ldapsearch -x -b 'ou=users,dc=conn-me,dc=ru' -s base '(uid=ikuchin)' namingContexts -D "cn=admin,dc=conn-me,dc=ru" -W -h ldap-service
```

```
618feb97 conn=2786 fd=15 ACCEPT from IP=192.168.48.2:35088 (IP=0.0.0.0:389)
618feb97 conn=2786 op=0 BIND dn="cn=admin,dc=conn-me,dc=ru" method=128
618feb97 conn=2786 op=0 BIND dn="cn=admin,dc=conn-me,dc=ru" mech=SIMPLE ssf=0
618feb97 conn=2786 op=0 RESULT tag=97 err=0 text=
618feb97 conn=2786 op=1 SRCH base="ou=users,dc=conn-me,dc=ru" scope=0 deref=0 filter="(uid=ikuchin)"
618feb97 conn=2786 op=1 SRCH attr=namingContexts
618feb97 conn=2786 op=1 SEARCH RESULT tag=101 err=0 nentries=0 text=
618feb97 conn=2786 op=2 UNBIND
618feb97 conn=2786 fd=15 closed
```

Line (5): SRCH base="ou=users,dc=conn-me,dc=ru"

#### Set SRCH scope to sub-tree: -s sub

```
ldapsearch -x -b 'ou=users,dc=conn-me,dc=ru' -s sub '(uid=ikuchin)' namingContexts -D "cn=admin,dc=conn-me,dc=ru" -W -h ldap-service
```

```
618fec2e conn=2787 fd=15 ACCEPT from IP=192.168.48.2:35106 (IP=0.0.0.0:389)
618fec2e conn=2787 op=0 BIND dn="cn=admin,dc=conn-me,dc=ru" method=128
618fec2e conn=2787 op=0 BIND dn="cn=admin,dc=conn-me,dc=ru" mech=SIMPLE ssf=0
618fec2e conn=2787 op=0 RESULT tag=97 err=0 text=
618fec2e conn=2787 op=1 SRCH base="ou=users,dc=conn-me,dc=ru" scope=2 deref=0 filter="(uid=ikuchin)"
618fec2e conn=2787 op=1 SRCH attr=namingContexts
618fec2e conn=2787 op=1 SEARCH RESULT tag=101 err=0 nentries=1 text=
618fec2e conn=2787 op=2 UNBIND
618fec2e conn=2787 fd=15 closed
```

Line (5): scope sets to 2 (sub-tree)

Line (7): 1 entry found

#### Example from SSH auth

```
618fe899 conn=2779 fd=18 ACCEPT from IP=192.168.168.12:57174 (IP=0.0.0.0:389)
618fe899 conn=2779 op=0 BIND dn="cn=admin,dc=conn-me,dc=ru" method=128
618fe899 conn=2779 op=0 BIND dn="cn=admin,dc=conn-me,dc=ru" mech=SIMPLE ssf=0
618fe899 conn=2779 op=0 RESULT tag=97 err=0 text=
618fe899 conn=2779 op=1 SRCH base="dc=conn-me,dc=ru" scope=2 deref=0 filter="(uid=ikuchin)"
618fe899 conn=2779 op=1 SEARCH RESULT tag=101 err=0 nentries=1 text=
618fe899 conn=2779 op=2 BIND anonymous mech=implicit ssf=0
618fe899 conn=2779 op=2 BIND dn="cn=Ivan Kuchin,ou=users,dc=conn-me,dc=ru" method=128
618fe899 conn=2779 op=2 BIND dn="cn=Ivan Kuchin,ou=users,dc=conn-me,dc=ru" mech=SIMPLE ssf=0
618fe899 conn=2779 op=2 RESULT tag=97 err=0 text=
618fe899 conn=2779 op=3 BIND anonymous mech=implicit ssf=0
618fe899 conn=2779 op=3 BIND dn="cn=admin,dc=conn-me,dc=ru" method=128
618fe899 conn=2779 op=3 BIND dn="cn=admin,dc=conn-me,dc=ru" mech=SIMPLE ssf=0
618fe899 conn=2779 op=3 RESULT tag=97 err=0 text=
618fe899 conn=2779 op=4 UNBIND
618fe899 conn=2779 fd=18 closed
```

#### Example from failed PHPMYADMIN auth

```
618fed19 conn=2790 fd=15 ACCEPT from IP=192.168.168.12:57302 (IP=0.0.0.0:389)
618fed19 conn=2790 op=0 BIND dn="uid=ikuchin,ou=users,dc=conn-me,dc=ru" method=128
618fed19 conn=2790 op=0 RESULT tag=97 err=49 text=
618fed19 conn=2790 op=1 UNBIND
618fed19 conn=2790 fd=15 closed
```

Line (2): unusual bind request DN. In normal operation first bind request uses admin's OU and in SRCH operation search request. But here it uses search request in BIND operation.

## FreeRADIUS installation

1\)  Pull out container

```
sudo docker pull freeradius/freeradius-server:latest
```

&#x20;2\)  Get a copy of /etc/freeradius from the container

a.  Run a container with local folder mounted &#x20;

```
sudo docker -it --rm -v ./123:/123 freeradius/freeradius-server:latest /bin/bash
```

b.  Copy /etc/freeradius to /123 keeping owner and access rights cp -rp /etc/freeradius /123

c.  Exit, container will be removed (due to --rm flag)

3\)  Add FreeRADIUS to the LDAP docker-compose

```docker
  radius:
    container_name: radius-service
    image: freeradius/freeradius-server:latest
    ports:
      - "1812:1812/udp"
      - "1813:1813/udp"
    environment:
      TZ: 'America/New_York'
    # Volumes store your data between container upgrades
    volumes:
      - './freeradius/etc/freeradius:/etc/freeradius'

    restart: unless-stopped
    command: freeradius -X
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
```

{% hint style="info" %}
Start-up command has -X flag, which produce debug info. Useful during initial config
{% endhint %}

4\) Add clients from RFC1918 to local copy of /etc/freeradius/clients.conf

```
client RFC1918_net1 {
        ipaddr          = 192.168.0.0/16
        secret          = __PASS__
}
client RFC1918_net2 {
        ipaddr          = 172.16.0.0/12
        secret          = __PASS__
}
client RFC1918_net3 {
        ipaddr          = 10.0.0.0/8
        secret          = __PASS__
}
```

5\) Add test user with RADIUS attribute to local copy of /etc/freeradius/users

```
bob     Cleartext-Password := "hello"
        Reply-Message := "Hello, %{User-Name} !",
        Service-Type := Administrative-User
```

{% hint style="info" %}
Service-type attribute added explicitly to user account
{% endhint %}

6\) To test RADIUS authentication run radius-side-container in the same docker-compose.yaml

```
  debug:
    container_name: debug
    image: freeradius/freeradius-server:latest
    restart: unless-stopped
    command: /bin/bash -c "sleep 3600"
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
```

7\) Run radtest inside debug-container

```
sudo docker exec -it debug /bin/bash -c “radtest bob hello radius-service 0 123Melanie123”
```

```
Sent Access-Request Id 252 from 0.0.0.0:59731 to 192.168.48.3:1812 length 73
        User-Name = "bob"
        User-Password = "hello"
        NAS-IP-Address = 192.168.48.3
        NAS-Port = 0
        Message-Authenticator = 0x00
        Cleartext-Password = "hello"
Received Access-Accept Id 252 from 192.168.48.3:1812 to 192.168.48.3:59731 length 40
        Reply-Message = "Hello, bob !"
        Service-Type = Administrative-User
```

Line (8): Access-Accept

Line (10): Service-Type attribute returned back

## Integration of FreeRadious and OpenLDAP

### **Simple option: with no LDAP attributes send back to RADIUS**

This option is convenieint when RADIUS or LDAP devices must use same user-account for authentication.

1\)      Edit local copy of FreeRADIUS LDAP config module \~/docker/openldap/freeradius/etc/freeradius/mods-available

```
ldap {
        server = 'ldap-service'
        identity = 'cn=admin,dc=conn-me,dc=ru'
        password = __PASSWORD__
        base_dn = 'dc=conn-me,dc=ru'
```

2\) Enable ldap conf-module

```
cd  ~/docker/openldap/freeradius/etc/freeradius/mods-enabled
ln -s ../mods-available/ldap ldap
```

3\) Enable ldap-authentication in file \~/docker/openldap/freeradius/etc/freeradius/sites-enabled

```
       Auth-Type LDAP {
               ldap
       }
```

4\) Test RADIUS-authC from LDAP account

```
sudo docker exec -it debug /bin/bash -c "radtest bob hello radius-service 0 __PASSWORD__"
```

```
Sent Access-Request Id 169 from 0.0.0.0:58913 to 192.168.48.3:1812 length 73
        User-Name = "bob"
        User-Password = "hello"
        NAS-IP-Address = 192.168.48.2
        NAS-Port = 0
        Message-Authenticator = 0x00
        Cleartext-Password = "hello"
Received Access-Accept Id 169 from 192.168.48.3:1812 to 192.168.48.2:58913 length 40
```

### **More complex: add LDAP attribute and map it to RADIUS attribute**

{% hint style="info" %}
Use case: WLC requires RADIUS attribute (Service-Type=Administrative) for admin user to authenticate and have read-write rights.
{% endhint %}

![](../.gitbook/assets/ldap\_and\_radius.png)

To force FreeRADIUS to send attribute use local user as described above or

1. Create custom schema in LDAP
2. Configure mapping from LDAP attributes to RADIUS attributes

&#x20;**Create custom LDAP schema.**

{% hint style="info" %}
/etc/ldap/schema must be mounted as a volume to the LDAP-container.
{% endhint %}

Download LDAP radius-schema from [github](https://github.com/FreeRADIUS/freeradius-server/tree/master/doc/schemas/ldap/openldap)

![](../.gitbook/assets/freeradius\_scheda\_github.png)

Convert “.schema” to “.ldif”  ([https://www.lisenet.com/2015/convert-openldap-schema-to-ldif/](https://www.lisenet.com/2015/convert-openldap-schema-to-ldif/))

In the OpenLDAP container:

```
more schema_conv.conf
```

Expected output:

```
include /etc/ldap/schema/freeradius.schema
```

```
mkdir /tmp/ldif
slaptest -f ./schema_conv.conf -F /tmp/ldif/
```

In the file /tmp/ldif/cn=config/cn=schema/cn={0}freeradius.ldif change following:

```
dn: cn={5}mail
objectClass: olcSchemaConfig
cn: {5}mail
```

To:

```
dn: cn=mail,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: mail
```

Delete these lines from the bottom:

```
structuralObjectClass: olcSchemaConfig
entryUUID: d53d1a8c-4261-1034-9085-738a9b3f3783
creatorsName: cn=config
createTimestamp: 20150206153742Z
entryCSN: 20150206153742.072733Z#000000#000#000000
modifiersName: cn=config
modifyTimestamp: 20150206153742Z
```

copy /tmp/ldif/cn\\=config/cn\\=schema/cn\\=\\{0\\}freeradius.ldif to /etc/ldap/freeradius.ldif

Add freeradius.ldif to LDAP-server:

```
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/freeradius.ldif
```

Verify successful add:

```
ls -1 /etc/ldap/slapd.d/cn\=config/cn\=schema
```

> cn={0}core.ldif
>
> cn={1}cosine.ldif
>
> cn={2}nis.ldif
>
> cn={3}inetorgperson.ldif
>
> cn={4}<mark style="background-color:yellow;">freeradius.ldif</mark>
>
> cn={5}mail.ldif
>
> cn={6}samba.ldif

**Add freeradius object schema to users DN**

![](<../.gitbook/assets/add\_radius\_obj\_schema\_to\_user (1).png>)

![](../.gitbook/assets/add\_radius\_obj\_schema\_to\_user\_2.png)

![](../.gitbook/assets/add\_radius\_obj\_schema\_to\_user\_3.png)

![](../.gitbook/assets/add\_radius\_obj\_schema\_to\_user\_4.png)

**Mapping LDAP attribute to RADIUS attribute**

Find the correct LDAP attribute value in standard RADIUS dictionaries:

```
egrep "\sService-Type\s" /usr/share/freeradius/* | grep Admin
```

```
/usr/share/freeradius/dictionary.rfc2865:VALUE  Service-Type          Administrative-User     6
/usr/share/freeradius/dictionary.rfc2865:VALUE  Service-Type        Callback-Administrative 11
```

We are interested in Line (1) because of Administrative-User

Then map LDAP attribute RADIUS in the local copy of /etc/raddb/mods-enabled/ldap

```
        update {
                control:Password-With-Header    += 'userPassword'
                reply:Service-Type              := 'radiusServiceType'
```

Test the implementation:

```
sudo docker exec -it debug /bin/bash -c "radtest ikuchin __USER_PASSWORD__ radius-service 0 __RADIUS_PASSWORD__"         
```

```
Sent Access-Request Id 212 from 0.0.0.0:58400 to 192.168.48.3:1812 length 77
        User-Name = "ikuchin"
        User-Password = "__user_password__"
        NAS-IP-Address = 192.168.48.2
        NAS-Port = 0
        Message-Authenticator = 0x00
        Cleartext-Password = "__User_Password__"
Received Access-Accept Id 212 from 192.168.48.3:1812 to 192.168.48.2:58400 length 26
        Service-Type = Administrative-User
```

Line (9) means that goal been achieved.

## RADIUS client config

Option 1 (preferred):

```
aaa new-model
!
radius server DEV
 address ipv4 192.168.168.12 auth-port 1812 acct-port 1813
 key 123Melanie123
aaa group server radius DEV
 server name DEV
 ip radius source-interface Loopback0
!
aaa authentication login DEV group DEV none
!
line vty 0 15
 login authentication DEV
```

Option 2:

```
aaa new-model
!
!
aaa group server radius DEV
 server-private 192.168.168.12 auth-port 1812 acct-port 1813 key 123Melanie123
 ip radius source-interface Loopback0
!
aaa authentication login DEV group DEV none
!
line vty 0 15
 login authentication DEV

```

