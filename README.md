# lnmp vagrant box
php7, nginx, mysql57

## Install

```
$ vagrant plugin install vagrant-bindfs
$ vagrant plugin install vagrant-hostmanager
$ vagrant up
```

_If you want to import db during provision you need to put dump.sql or dump.sql.gz into folder near vagrant._