---
layout: article
title: Jupyterhub的LDAP打通OSS.md
tags: Jupyterhub
category: blog
date: 2022-07-27 17:16:00 +08:00
mermaid: true
---
官方文档：[https://github.com/jupyterhub/ldapauthenticator](https://github.com/jupyterhub/ldapauthenticator)

```bash
cd /usr/local/lib/python3.6/site-packages/ldapauthenticator
vim ldapauthenticator.py 
```
:312
```bash
             self.server_address, port=self.server_port, use_ssl=self.use_ssl
         )
         auto_bind = (
-            ldap3.AUTO_BIND_NO_TLS if self.use_ssl else ldap3.AUTO_BIND_TLS_BEFORE_BIND
+            ldap3.AUTO_BIND_NO_TLS if not self.use_ssl else ldap3.AUTO_BIND_TLS_BEFORE_BIND
         )
         conn = ldap3.Connection(
             server, user=userdn, password=password, auto_bind=auto_bind
```
             

```bash
from subprocess import check_call


def pre_spawn_hook(spawner):
    username = spawner.user.name
    try:
        check_call(['useradd', '-ms', '/bin/bash', username])
    except Exception as e:
        print(f'{e}')
c.Spawner.pre_spawn_hook = pre_spawn_hook
c.JupyterHub.authenticator_class = 'ldapauthenticator.LDAPAuthenticator'
c.LDAPAuthenticator.server_address =  'ldap://  '
c.LDAPAuthenticator.lookup_dn = True
c.LDAPAuthenticator.use_ssl=False
c.LDAPAuthenticator.lookup_dn_search_user = 'cn=ldap_admin@yutao.co,ou=user,dc=yutao,dc=com'
c.LDAPAuthenticator.lookup_dn_search_password = '*****'
c.LDAPAuthenticator.user_search_base = 'ou=user,dc=yutao,dc=com'
c.LDAPAuthenticator.user_attribute = 'sn'
c.LDAPAuthenticator.lookup_dn_user_dn_attribute='sn'
```

