# fail2ban

## Initial configuration

```
sudo apt install fail2ban
```

Configuration folder /etc/fail2ban. Enable following jails:

```
sudo fail2ban-client start sshd
sudo fail2ban-client start sshd-ddos
sudo fail2ban-client start apache-auth
sudo fail2ban-client start apache-noscript
sudo fail2ban-client start apache-overflows
sudo fail2ban-client start apache-nohome
sudo fail2ban-client start apache-botsearch
sudo fail2ban-client start apache-fakegooglebot
sudo fail2ban-client start vsftpd
```

Monitoring Fail2ban:

Bans today per IP:

```
grep "Ban " /var/log/fail2ban.log | grep `date +%Y-%m-%d` | awk '{print $NF}' | sort | awk '{print $1,"("$1")"}' | logresolve | uniq -c | sort –n
```

> 7 home (72.93.214.69)

Bans since the Earth per day per section

```
zgrep -h "Ban " /var/log/fail2ban.log* | awk '{print $6,$1}' | sort | uniq -c 
```

> 7 \[apache-auth] 2017-03-08
>
> 3 \[sshd] 2017-03-05

## Add your own filter

Folder filter.d contains all filters. Every filter is set of regexps. You can tets your filter one of the following ways:

i.      String to match, failregex

```
fail2ban-regex '60.191.38.77 - - [17/Sep/2019:11:59:22 +0300] "GET / HTTP/1.1" 403 7449 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:47.0) Gecko/20100101 Firefox/47.0"' '^<HOST> .*Firefox'
```

ii. File instead log message

```
fail2ban-regex /home/httpd/images.timecard.ru/logs/access.log '^<HOST> .*Firefox'
```

iii. File filter.d

```
fail2ban-regex /home/httpd/images.timecard.ru/logs/access.log /etc/fail2ban/filter.d/timecard-bruteforce.conf
```

This filter looking for wrong filenames.

```
ikuchin@www:/etc/fail2ban/jail.d$ more ../filter.d/timecard-bruteforce.conf 

[Definition]
folders = (invoices_|agreements_|helpdesk_ticket_attaches|smartway_vouchers|templates_)

failregex = ^<HOST> -.*"(GET|POST|HEAD) /%(folders)s.*HTTP/\d\.\d" 404

ignoreregex =
```

## Add your own jail

```
[timecard-bruteforce]
enabled = true
filter = timecard-bruteforce
action = iptables-multiport[name=timecard-bruteforce, port="80,443", protocol=tcp]
         sendmail-whois-lines[name=timecard-bruteforce, dest=xxxx@gmail.com, sender=xxxx@connme.ru, logpath=/var/log/fail2ban.log]
logpath  = /home/httpd/images.timecard.ru/logs/access.log
```

