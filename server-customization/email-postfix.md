# Email / Postfix

## DKIM installation / config

{% hint style="warning" %}
Read comment below whitepaper. Whitepaper is incorrect
{% endhint %}

{% embed url="https://tecadmin.net/setup-dkim-with-postfix-on-ubuntu-debian" %}

## DKIM allow external hosts

By default DKIM won't be signing messages sent by external hosts. To allow add `InternalHosts` to opendkim.conf

{% embed url="https://easyengine.io/tutorials/mail/setup-opendkim" %}

## Mail tester

{% embed url="https://www.appmaildev.com/en/dkim" %}

## Sending message from custom domain

Prepare text file with message body:

```
From: <info@timecard.ru>
Subject: Автоматическое сообщение: ошибка загрузки фаила
Content-Type: text/plain; charset=utf-8

Добрый день,

   Автоматическое сообщение:
Формат одного из ......
```

```
sendmail recipient@domain.com < mail.txt
```

