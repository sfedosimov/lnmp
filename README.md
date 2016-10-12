# lnmp vagrant box
php7, nginx, mysql57

## Install

```
$ vagrant plugin install vagrant-bindfs
$ vagrant plugin install vagrant-hostmanager
$ vagrant up
```

_If you want to import db during provision you need to put dump.sql or dump.sql.gz into folder near vagrant._

## Credentials
**IP**: 192.168.192.168
**DOMAIN**: site.dev
**DBUSER**: root
**DBPASSWORD**: password1
**EMAIL TO TEST**: vagrant@site.dev (see mails in /var/mail/vagrant)