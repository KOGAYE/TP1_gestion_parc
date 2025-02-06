# I.service ssh

## 1. analyse du service

S'assurer que le service sshd est démarré
```bash 
[kogaye@web ~]$ systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Sat 2024-12-21 21:40:38 CET; 7min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 717 (sshd)
      Tasks: 1 (limit: 11084)
     Memory: 2.8M
        CPU: 68ms
     CGroup: /system.slice/sshd.service
```

analyser les processus liés au service ssh
```bash
root         717       1  0 Dec21 ?        00:00:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root        1271     717  0 Dec21 ?        00:00:00 sshd: kogaye [priv]
kogaye      1275    1271  0 Dec21 ?        00:00:00 sshd: kogaye@pts/0
kogaye      1334    1276  0 00:06 pts/0    00:00:00 grep --color=auto sshd
```



Déterminer le port sur lequel écoute le service ssh

```bash
[kogaye@web ~]$ sudo ss -tlnp
[sudo] password for kogaye:
State     Recv-Q    Send-Q         Local Address:Port         Peer Address:Port    Process
LISTEN    0         128                  0.0.0.0:22                0.0.0.0:*        users:(("sshd",pid=717,fd=3))
LISTEN    0         128                     [::]:22                   [::]:*        users:(("sshd",pid=717,fd=4))
```

port--> 22


```bash
[kogaye@web ~]$ journalctl -u sshd
Dec 21 22:21:24 web.tp1.b1 systemd[1]: Starting OpenSSH server daemon...
Dec 21 22:21:24 web.tp1.b1 sshd[717]: Server listening on 0.0.0.0 port 22.
Dec 21 22:21:24 web.tp1.b1 sshd[717]: Server listening on :: port 22.
Dec 21 22:21:24 web.tp1.b1 systemd[1]: Started OpenSSH server daemon.
Dec 21 22:32:53 web.tp1.b1 sshd[1271]: Accepted password for kogaye from 10.1.1.3 port 65387 ssh2
Dec 21 22:32:53 web.tp1.b1 sshd[1271]: pam_unix(sshd:session): session opened for user kogaye(uid=1000) by kogaye(ui>
```

identifier le fichier de config de ssh
```bash
[kogaye@web ~]$ sudo cat /etc/ssh/sshd_config
[sudo] password for kogaye:
#       $OpenBSD: sshd_config,v 1.104 2021/07/02 22:55:21 dtucker Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

# To modify the system-wide sshd configuration, create a  *.conf  file under
#  /etc/ssh/sshd_config.d/  which will be automatically included below
Include /etc/ssh/sshd_config.d/*.conf

# If you want to change the port on a SELinux system, you have to tell
# SELinux about this change.
# semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
#
#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
#PermitRootLogin prohibit-password
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# The default is to check both .ssh/authorized_keys and .ssh/authorized_keys2
# but this is overridden so installations will only check .ssh/authorized_keys
AuthorizedKeysFile      .ssh/authorized_keys

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no

# Change to no to disable s/key passwords
#KbdInteractiveAuthentication yes

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no
#KerberosUseKuserok yes

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no
#GSSAPIEnablek5users no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the KbdInteractiveAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via KbdInteractiveAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and KbdInteractiveAuthentication to 'no'.
# WARNING: 'UsePAM no' is not supported in RHEL and may cause several
# problems.
#UsePAM no

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
#X11Forwarding no
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
#PrintMotd yes
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /var/run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# override default of no subsystems
Subsystem       sftp    /usr/libexec/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server
```

modifier le fichier de conf

```bash
[kogaye@web ~]$ echo $RANDOM
11452


[kogaye@web ~]$ sudo cat /etc/ssh/sshd_config | grep Port
Port 11452
```


```bash
[kogaye@web ~]$ sudo firewall-cmd --list-all | grep 11452
  ports: 11452/tcp
```


redémarrer le service 
```bash
[kogaye@web ~]$ sudo systemctl restart sshd
```

connection au port spécifique
```bash
PS C:\Users\kogaye> ssh kogaye@10.1.1.1 -p 11452
kogaye@10.1.1.1's password:
Last login: Sat Dec 21 23:10:25 2024 from 10.1.1.3
[kogaye@web ~]$
```


# II. Service HTTP

## 1. Mise en place

installer le serveur NGINX

```bash
[kogaye@web ~]$ sudo dnf install nginx
[sudo] password for kogaye:
Rocky Linux 9 - BaseOS                                                                12 kB/s | 4.1 kB     00:00
Rocky Linux 9 - AppStream                                                             18 kB/s | 4.5 kB     00:00
Rocky Linux 9 - Extras                                                                11 kB/s | 2.9 kB     00:00
Dependencies resolved.
=====================================================================================================================
 Package                        Architecture        Version                             Repository              Size
=====================================================================================================================
Installing:
 nginx                          x86_64              2:1.20.1-20.el9.0.1                 appstream               36 k
Installing dependencies:
 nginx-core                     x86_64              2:1.20.1-20.el9.0.1                 appstream              566 k
 nginx-filesystem               noarch              2:1.20.1-20.el9.0.1                 appstream              8.4 k
 rocky-logos-httpd              noarch              90.15-2.el9                         appstream               24 k

Transaction Summary
=====================================================================================================================
Install  4 Packages

Total download size: 634 k
Installed size: 1.8 M
Is this ok [y/N]: y
Downloading Packages:
(1/4): nginx-filesystem-1.20.1-20.el9.0.1.noarch.rpm                                  93 kB/s | 8.4 kB     00:00
(2/4): rocky-logos-httpd-90.15-2.el9.noarch.rpm                                      258 kB/s |  24 kB     00:00
(3/4): nginx-1.20.1-20.el9.0.1.x86_64.rpm                                            365 kB/s |  36 kB     00:00
(4/4): nginx-core-1.20.1-20.el9.0.1.x86_64.rpm                                       3.2 MB/s | 566 kB     00:00
---------------------------------------------------------------------------------------------------------------------
Total                                                                                1.3 MB/s | 634 kB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                             1/1
  Running scriptlet: nginx-filesystem-2:1.20.1-20.el9.0.1.noarch                                                 1/4
  Installing       : nginx-filesystem-2:1.20.1-20.el9.0.1.noarch                                                 1/4
  Installing       : nginx-core-2:1.20.1-20.el9.0.1.x86_64                                                       2/4
  Installing       : rocky-logos-httpd-90.15-2.el9.noarch                                                        3/4
  Installing       : nginx-2:1.20.1-20.el9.0.1.x86_64                                                            4/4
  Running scriptlet: nginx-2:1.20.1-20.el9.0.1.x86_64                                                            4/4
  Verifying        : rocky-logos-httpd-90.15-2.el9.noarch                                                        1/4
  Verifying        : nginx-filesystem-2:1.20.1-20.el9.0.1.noarch                                                 2/4
  Verifying        : nginx-2:1.20.1-20.el9.0.1.x86_64                                                            3/4
  Verifying        : nginx-core-2:1.20.1-20.el9.0.1.x86_64                                                       4/4

Installed:
  nginx-2:1.20.1-20.el9.0.1.x86_64                             nginx-core-2:1.20.1-20.el9.0.1.x86_64
  nginx-filesystem-2:1.20.1-20.el9.0.1.noarch                  rocky-logos-httpd-90.15-2.el9.noarch

Complete!
```



Démarrer le service NGINX

```bash
[kogaye@web ~]$ sudo systemctl start nginx
```


Déterminer sur quel port tourne NGINX
```bash
[kogaye@web ~]$ sudo ss -tlnp | grep nginx
LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=1466,fd=6),("nginx",pid=1465,fd=6))
LISTEN 0      511             [::]:80           [::]:*    users:(("nginx",pid=1466,fd=7),("nginx",pid=1465,fd=7))
```


firewall

```bash
[kogaye@web ~]$ sudo firewall-cmd --permanent --add-port=80/tcp
success
[kogaye@web ~]$ sudo firewall-cmd --reload
success
```

Déterminer le processus lié au service NGINX

```bash
[kogaye@web ~]$ ps -ef | grep nginx
root        1465       1  0 12:05 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx       1466    1465  0 12:05 ?        00:00:00 nginx: worker process
```

Déterminer le nom de l'utilisateur que lance nginx

```bash
[kogaye@web ~]$ sudo cat /etc/passwd | grep nginx
nginx:x:996:993:Nginx web server:/var/lib/nginx:/sbin/nologin
```
l'utilisateur qui lance nginx est nginx


test
```git bash
$ curl http://10.1.1.1:80 | head
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  7620  100  7620<!doctype html>  0      0 --:--:-- --:--:-- --:--:--     0
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
100  7620  100  7620    0     0   649k      0 --:--:-- --:--:-- --:--:--  676k
curl: Failed writing body
```


## 2 Analyser la conf de NGINX

Déterminer le path du fichier de configuration de NGINX
```bash
[kogaye@web ~]$ ls -al /etc/nginx/nginx.conf
-rw-r--r--. 1 root root 2334 Dec  26 22:35 /etc/nginx/nginx.conf
```

trouver dans le fichier conf

```bash
[kogaye@web ~]$ cat /etc/nginx/nginx.conf | grep 80 -A 14
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
```

```bash
[kogaye@web ~]$ cat /etc/nginx/nginx.conf | grep "include /usr/"
include /usr/share/nginx/modules/*.conf;
```

```bash 
[kogaye@web ~]$ cat /etc/nginx/nginx.conf | grep "user n"
user nginx;
```


```bash
[kogaye@web conf.d]$ echo $RANDOM
5361
[kogaye@web conf.d]$ sudo nano site1_tp1
[kogaye@web conf.d]$ sudo firwall-cmd --permanent --remove-port=80/tcp
sudo: firwall-cmd: command not found
[kogaye@web conf.d]$ sudo firewall-cmd --permanent --remove-port=80/tcp
success
[kogaye@web conf.d]$ sudo firewall-cmd --permanent --add-port=5361/tcp
success
```


fichier config nginx 

```bash

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
}
           
```


```bash
[kogaye@web default.d]$ echo $RANDOM
181768
```


```bash
  GNU nano 5.6.1            /etc/nginx/default.d/tp1_conf.conf                       server {
  # le port choisi devra être obtenu avec un 'echo $RANDOM' là encore
  listen 18168;

  root /var/www/tp1_parc;
}
```

```bash
$ curl 10.1.1.1:18168
<h1> MEOW mon premier serveur web <h1>

```



# III.Monitoring et alerting



```bash
[kogaye@monitoring ~]$ sudo ss -tlnp | grep netdata
LISTEN 0      4096       127.0.0.1:8125       0.0.0.0:*    users:(("netdata",pid=2966,fd=46))
LISTEN 0      4096         0.0.0.0:19999      0.0.0.0:*    users:(("netdata",pid=2966,fd=6))
LISTEN 0      4096           [::1]:8125          [::]:*    users:(("netdata",pid=2966,fd=45))
LISTEN 0      4096            [::]:19999         [::]:*    users:(("netdata",pid=2966,fd=7))
[kogaye@monitoring ~]$ sudo firewall-cmd --permanent --add-port=19999/tcp
success
[kogaye@monitoring ~]$ sudo firewall-cmd --reload
success
```

Donc netdata écoute sur 19999


```bash
Kogaye@Kogaye MINGW64 ~
$ curl http://10.1.1.2:19999 | head 7
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0head: cannot open '7' for reading: No such file or directory
 63  106k   63 69709    0     0  1857k      0 --:--:-- --:--:-- --:--:-- 1890k
curl: (23) Failure writing output to destination, passed 16384 returned 4019
```



```bash

jobs:
  - name: WEB.web.tp1.b1
    host: 10.1.1.1
    ports:
      - 18178

  - name: SSH.web.tp1.b1
    host: 10.1.1.1
    ports:
      - 22


```
