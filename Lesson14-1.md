# Creating 2 LXC in VM with names node1 & node2 with access via ssh keys
```sh
root@TEST1:~/.ssh# lxc-create -t debian -n node1
root@TEST1:~/.ssh# lxc-create -t debian -n node2
root@TEST1:~/.ssh# echo "PermitRootLogin yes" >> /var/lib/lxc/node1/rootfs/etc/ssh/sshd_config
root@TEST1:~/.ssh# echo "PermitRootLogin yes" >> /var/lib/lxc/node2/rootfs/etc/ssh/sshd_config
root@TEST1:~/.ssh# lxc-start -n node1
root@TEST1:~/.ssh# lxc-start -n node2
root@TEST1:~/.ssh# lxc-info -n node1 | grep IP | awk {'print $2'}
10.0.3.101
root@TEST1:~/.ssh# lxc-info -n node2 | grep IP | awk {'print $2'}
10.0.3.141
root@TEST1:~/.ssh# lxc-attach -n node1
root@node1:/# passwd
New password:
Retype new password:
passwd: password updated successfully
root@node1:/# exit
exit
root@TEST1:~/.ssh# lxc-attach -n node2
root@node2:/# passwd
New password:
Retype new password:
passwd: password updated successfully
root@node2:/# exit
exit
root@TEST1:~/.ssh# ssh-keygen -t ed25519 -C "For LXC" -f lxc_id25519
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in lxc_id25519
Your public key has been saved in lxc_id25519.pub
The key fingerprint is:
SHA256:HSKubp9VJndK77WUXhdKNrc4DDCcfhWasLbQKpoE8Vw For LXC
The key's randomart image is:
+--[ED25519 256]--+
| .   E   .   .   |
|  + .   o + o .  |
| . o  ...O.o .   |
|  .  . .=o+..    |
|   . ...So*o.+ o |
|  . o..  *.+= =.o|
|   o.   . . .=+.o|
|   ..  o   . +.o.|
|   ...o     . o  |
+----[SHA256]-----+
root@TEST1:~/.ssh# chmod 600 lxc*
root@TEST1:~/.ssh# ssh-copy-id -i ./lxc_id25519.pub root@10.0.3.101 && ssh-copy-id -i ./lxc_id25519.pub root@10.0.3.141
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "./lxc_id25519.pub"
The authenticity of host '10.0.3.101 (10.0.3.101)' can't be established.
ECDSA key fingerprint is SHA256:Py6npY7Xq5ygTzOSML+zM295+j4Mnd5mB5KgtHbpigg.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@10.0.3.101's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@10.0.3.101'"
and check to make sure that only the key(s) you wanted were added.

/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "./lxc_id25519.pub"
The authenticity of host '10.0.3.141 (10.0.3.141)' can't be established.
ECDSA key fingerprint is SHA256:ZJRRK06DfIWWGQKvPf32exLCguIBTgrtHGNmPw1SCps.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@10.0.3.141's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@10.0.3.141'"
and check to make sure that only the key(s) you wanted were added.

root@TEST1:~/.ssh# touch config && chmod 600 ./config
root@TEST1:~/.ssh# cat <<EOF > config
host node1
 Hostname 10.0.3.101
 user root
 port 22
 IdentityFile /root/.ssh/lxc_id25519

host node2
 Hostname 10.0.3.141
 user root
 port 22
 IdentityFile /root/.ssh/lxc_id25519
EOF
root@TEST1:~/.ssh# scp config node1:.ssh/
config                                                                                                                100%  161   283.8KB/s   00:00
root@TEST1:~/.ssh# scp lxc_id25519 node1:.ssh/
lxc_id25519                                                                                                           100%  399   686.0KB/s   00:00
root@TEST1:~/.ssh# scp config node2:.ssh/
config                                                                                                                100%  161   263.2KB/s   00:00
root@TEST1:~/.ssh# scp lxc_id25519 node2:.ssh/
lxc_id25519                                                                                                           100%  399   689.8KB/s   00:00
root@TEST1:~/.ssh# ssh node1
Linux node1 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb  2 11:43:01 2023 from 10.0.3.1
root@node1:~# ssh node2
The authenticity of host '10.0.3.141 (10.0.3.141)' can't be established.
ECDSA key fingerprint is SHA256:Py6npY7Xq5ygTzOSML+zM295+j4Mnd5mB5KgtHbpigg.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.3.141' (ECDSA) to the list of known hosts.
Linux node2 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb  2 11:41:43 2023 from 10.0.3.101
root@node2:~# ssh node1
The authenticity of host '10.0.3.101 (10.0.3.101)' can't be established.
ECDSA key fingerprint is SHA256:Py6npY7Xq5ygTzOSML+zM295+j4Mnd5mB5KgtHbpigg.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.3.101' (ECDSA) to the list of known hosts.
Linux node1 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb  2 11:43:31 2023 from 10.0.3.1
root@node1:~#

```
# Creating backup_logs and synhronization directory /var/log/apt exclude term.log with rsync
```sh
root@node1:~# apt-get update && apt-get install rsync -y
Get:1 http://security.debian.org/debian-security stable-security InRelease [48.4 kB]
Hit:2 http://deb.debian.org/debian stable InRelease
Get:3 http://security.debian.org/debian-security stable-security/main amd64 Packages [222 kB]
Get:4 http://security.debian.org/debian-security stable-security/main Translation-en [145 kB]
Get:5 http://deb.debian.org/debian stable/main Translation-en [6,240 kB]
Fetched 538 kB in 2s (308 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
The following additional packages will be installed:
  libpopt0
Suggested packages:
  python3
The following NEW packages will be installed:
  libpopt0 rsync
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 446 kB of archives.
After this operation, 987 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian stable/main amd64 libpopt0 amd64 1.18-2 [49.6 kB]
Get:2 http://deb.debian.org/debian stable/main amd64 rsync amd64 3.2.3-4+deb11u1 [396 kB]
Fetched 446 kB in 0s (1,607 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libpopt0:amd64.
(Reading database ... 9089 files and directories currently installed.)
Preparing to unpack .../libpopt0_1.18-2_amd64.deb ...
Unpacking libpopt0:amd64 (1.18-2) ...
Selecting previously unselected package rsync.
Preparing to unpack .../rsync_3.2.3-4+deb11u1_amd64.deb ...
Unpacking rsync (3.2.3-4+deb11u1) ...
Setting up libpopt0:amd64 (1.18-2) ...
Setting up rsync (3.2.3-4+deb11u1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/rsync.service → /lib/systemd/system/rsync.service.
Processing triggers for libc-bin (2.31-13+deb11u5) ...
root@node1:~# ls -la
total 24
drwx------  3 root root 4096 Feb  2 11:12 .
drwxr-xr-x 18 root root 4096 Feb  2 11:09 ..
-rw-------  1 root root  131 Feb  2 11:43 .bash_history
-rw-r--r--  1 root root  571 Apr 10  2021 .bashrc
-rw-r--r--  1 root root  161 Jul  9  2019 .profile
drwx------  2 root root 4096 Feb  2 11:41 .ssh
root@node1:~# pwd
/root
root@node1:~# mkdir backup_logs
root@node1:~# exit
logout
Connection to 10.0.3.101 closed.
root@node2:~# apt-get update && apt-get install rsync -y
Get:1 http://security.debian.org/debian-security stable-security InRelease [48.4 kB]
Hit:2 http://deb.debian.org/debian stable InRelease
Get:3 http://deb.debian.org/debian stable/main Translation-en [6,240 kB]
Get:4 http://security.debian.org/debian-security stable-security/main amd64 Packages [222 kB]
Get:5 http://security.debian.org/debian-security stable-security/main Translation-en [145 kB]
Fetched 6,655 kB in 3s (2,506 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
The following additional packages will be installed:
  libpopt0
Suggested packages:
  python3
The following NEW packages will be installed:
  libpopt0 rsync
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 446 kB of archives.
After this operation, 987 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian stable/main amd64 libpopt0 amd64 1.18-2 [49.6 kB]
Get:2 http://deb.debian.org/debian stable/main amd64 rsync amd64 3.2.3-4+deb11u1 [396 kB]
Fetched 446 kB in 0s (2,780 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libpopt0:amd64.
(Reading database ... 9089 files and directories currently installed.)
Preparing to unpack .../libpopt0_1.18-2_amd64.deb ...
Unpacking libpopt0:amd64 (1.18-2) ...
Selecting previously unselected package rsync.
Preparing to unpack .../rsync_3.2.3-4+deb11u1_amd64.deb ...
Unpacking rsync (3.2.3-4+deb11u1) ...
Setting up libpopt0:amd64 (1.18-2) ...
Setting up rsync (3.2.3-4+deb11u1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/rsync.service → /lib/systemd/system/rsync.service.
Processing triggers for libc-bin (2.31-13+deb11u5) ...
root@node2:~# mkdir backup_logs
root@node2:~# rsync -a /var/log/apt --exclude='term.log' node1:/root/backup_logs
root@node2:~# ssh node1
Linux node1 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb  2 11:43:40 2023 from 10.0.3.141
root@node1:~# ls -la backup_logs/apt/
total 20
drwxr-xr-x 2 root root 4096 Feb  2 11:48 .
drwxr-xr-x 3 root root 4096 Feb  2 11:49 ..
-rw-r--r-- 1 root root 6200 Feb  2 11:48 eipp.log.xz
-rw-r--r-- 1 root root  178 Feb  2 11:48 history.log
root@node1:~# rsync -a /var/log/apt/ --exclude='term.log' node2:/root/backup_logs/
root@node1:~# exit
logout
Connection to 10.0.3.101 closed.
root@node2:~# ls -la backup_logs/
total 24
drwxr-xr-x 2 root root 4096 Feb  2 11:46 .
drwx------ 4 root root 4096 Feb  2 11:48 ..
-rw-r--r-- 1 root root 6200 Feb  2 11:46 eipp.log.xz
-rw-r--r-- 1 root root  178 Feb  2 11:46 history.log
```
# Setup cron on node2 only!
```sh
root@node2:~# apt-get install cron -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  ca-certificates exim4-base exim4-config exim4-daemon-light gsasl-common guile-2.2-libs libevent-2.1-7 libexpat1 libfribidi0 libgc1 libgnutls-dane0
  libgsasl7 libidn11 libldap-2.4-2 libldap-common libltdl7 libmailutils7 libmariadb3 libmpdec3 libntlm0 libpython3.9 libpython3.9-minimal
  libpython3.9-stdlib libreadline8 libsasl2-2 libsasl2-modules libsasl2-modules-db libsqlite3-0 libunbound8 mailutils mailutils-common mariadb-common
  media-types mysql-common openssl psmisc readline-common
Suggested packages:
  anacron logrotate checksecurity exim4-doc-html | exim4-doc-info eximon4 file spf-tools-perl swaks dns-root-data libsasl2-modules-gssapi-mit
  | libsasl2-modules-gssapi-heimdal libsasl2-modules-ldap libsasl2-modules-otp libsasl2-modules-sql mailutils-mh mailutils-doc readline-doc
The following NEW packages will be installed:
  ca-certificates cron exim4-base exim4-config exim4-daemon-light gsasl-common guile-2.2-libs libevent-2.1-7 libexpat1 libfribidi0 libgc1
  libgnutls-dane0 libgsasl7 libidn11 libldap-2.4-2 libldap-common libltdl7 libmailutils7 libmariadb3 libmpdec3 libntlm0 libpython3.9
  libpython3.9-minimal libpython3.9-stdlib libreadline8 libsasl2-2 libsasl2-modules libsasl2-modules-db libsqlite3-0 libunbound8 mailutils
  mailutils-common mariadb-common media-types mysql-common openssl psmisc readline-common
0 upgraded, 38 newly installed, 0 to remove and 0 not upgraded.
Need to get 19.3 MB of archives.
After this operation, 86.2 MB of additional disk space will be used.
Get:1 http://deb.debian.org/debian stable/main amd64 cron amd64 3.0pl1-137 [99.6 kB]
Get:2 http://deb.debian.org/debian stable/main amd64 readline-common all 8.1-1 [73.7 kB]
Get:3 http://deb.debian.org/debian stable/main amd64 libreadline8 amd64 8.1-1 [169 kB]
Get:4 http://deb.debian.org/debian stable/main amd64 openssl amd64 1.1.1n-0+deb11u3 [853 kB]
Get:5 http://deb.debian.org/debian stable/main amd64 ca-certificates all 20210119 [158 kB]
Get:6 http://deb.debian.org/debian stable/main amd64 media-types all 4.0.0 [30.3 kB]
Get:7 http://deb.debian.org/debian stable/main amd64 exim4-config all 4.94.2-7 [335 kB]
Get:8 http://deb.debian.org/debian stable/main amd64 exim4-base amd64 4.94.2-7 [1,175 kB]
Get:9 http://deb.debian.org/debian stable/main amd64 libevent-2.1-7 amd64 2.1.12-stable-1 [188 kB]
Get:10 http://deb.debian.org/debian stable/main amd64 libunbound8 amd64 1.13.1-1 [504 kB]
Get:11 http://deb.debian.org/debian stable/main amd64 libgnutls-dane0 amd64 3.7.1-5+deb11u2 [395 kB]
Get:12 http://deb.debian.org/debian stable/main amd64 libidn11 amd64 1.33-3 [116 kB]
Get:13 http://deb.debian.org/debian stable/main amd64 exim4-daemon-light amd64 4.94.2-7 [658 kB]
Get:14 http://deb.debian.org/debian stable/main amd64 gsasl-common all 1.10.0-4+deb11u1 [175 kB]
Get:15 http://deb.debian.org/debian stable/main amd64 libgc1 amd64 1:8.0.4-3 [239 kB]
Get:16 http://deb.debian.org/debian stable/main amd64 libltdl7 amd64 2.4.6-15 [391 kB]
Get:17 http://deb.debian.org/debian stable/main amd64 guile-2.2-libs amd64 2.2.7+1-6 [4,980 kB]
Get:18 http://deb.debian.org/debian stable/main amd64 libexpat1 amd64 2.2.10-2+deb11u5 [98.2 kB]
Get:19 http://deb.debian.org/debian stable/main amd64 libfribidi0 amd64 1.0.8-2+deb11u1 [64.9 kB]
Get:20 http://deb.debian.org/debian stable/main amd64 libntlm0 amd64 1.6-3 [84.7 kB]
Get:21 http://deb.debian.org/debian stable/main amd64 libgsasl7 amd64 1.10.0-4+deb11u1 [195 kB]
Get:22 http://deb.debian.org/debian stable/main amd64 libsasl2-modules-db amd64 2.1.27+dfsg-2.1+deb11u1 [69.1 kB]
Get:23 http://deb.debian.org/debian stable/main amd64 libsasl2-2 amd64 2.1.27+dfsg-2.1+deb11u1 [106 kB]
Get:24 http://deb.debian.org/debian stable/main amd64 libldap-2.4-2 amd64 2.4.57+dfsg-3+deb11u1 [232 kB]
Get:25 http://deb.debian.org/debian stable/main amd64 libldap-common all 2.4.57+dfsg-3+deb11u1 [95.8 kB]
Get:26 http://deb.debian.org/debian stable/main amd64 mailutils-common all 1:3.10-3 [728 kB]
Get:27 http://deb.debian.org/debian stable/main amd64 mysql-common all 5.8+1.0.7 [7,464 B]
Get:28 http://deb.debian.org/debian stable/main amd64 mariadb-common all 1:10.5.18-0+deb11u1 [36.9 kB]
Get:29 http://deb.debian.org/debian stable/main amd64 libmariadb3 amd64 1:10.5.18-0+deb11u1 [176 kB]
Get:30 http://deb.debian.org/debian stable/main amd64 libpython3.9-minimal amd64 3.9.2-1 [801 kB]
Get:31 http://deb.debian.org/debian stable/main amd64 libmpdec3 amd64 2.5.1-1 [87.7 kB]
Get:32 http://deb.debian.org/debian stable/main amd64 libsqlite3-0 amd64 3.34.1-3 [797 kB]
Get:33 http://deb.debian.org/debian stable/main amd64 libpython3.9-stdlib amd64 3.9.2-1 [1,684 kB]
Get:34 http://deb.debian.org/debian stable/main amd64 libpython3.9 amd64 3.9.2-1 [1,691 kB]
Get:35 http://deb.debian.org/debian stable/main amd64 libmailutils7 amd64 1:3.10-3+b1 [893 kB]
Get:36 http://deb.debian.org/debian stable/main amd64 libsasl2-modules amd64 2.1.27+dfsg-2.1+deb11u1 [104 kB]
Get:37 http://deb.debian.org/debian stable/main amd64 mailutils amd64 1:3.10-3+b1 [576 kB]
Get:38 http://deb.debian.org/debian stable/main amd64 psmisc amd64 23.4-2 [198 kB]
Fetched 19.3 MB in 3s (5,919 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package cron.
(Reading database ... 9162 files and directories currently installed.)
Preparing to unpack .../00-cron_3.0pl1-137_amd64.deb ...
Unpacking cron (3.0pl1-137) ...
Selecting previously unselected package readline-common.
Preparing to unpack .../01-readline-common_8.1-1_all.deb ...
Unpacking readline-common (8.1-1) ...
Selecting previously unselected package libreadline8:amd64.
Preparing to unpack .../02-libreadline8_8.1-1_amd64.deb ...
Unpacking libreadline8:amd64 (8.1-1) ...
Selecting previously unselected package openssl.
Preparing to unpack .../03-openssl_1.1.1n-0+deb11u3_amd64.deb ...
Unpacking openssl (1.1.1n-0+deb11u3) ...
Selecting previously unselected package ca-certificates.
Preparing to unpack .../04-ca-certificates_20210119_all.deb ...
Unpacking ca-certificates (20210119) ...
Selecting previously unselected package media-types.
Preparing to unpack .../05-media-types_4.0.0_all.deb ...
Unpacking media-types (4.0.0) ...
Selecting previously unselected package exim4-config.
Preparing to unpack .../06-exim4-config_4.94.2-7_all.deb ...
Unpacking exim4-config (4.94.2-7) ...
Selecting previously unselected package exim4-base.
Preparing to unpack .../07-exim4-base_4.94.2-7_amd64.deb ...
Unpacking exim4-base (4.94.2-7) ...
Selecting previously unselected package libevent-2.1-7:amd64.
Preparing to unpack .../08-libevent-2.1-7_2.1.12-stable-1_amd64.deb ...
Unpacking libevent-2.1-7:amd64 (2.1.12-stable-1) ...
Selecting previously unselected package libunbound8:amd64.
Preparing to unpack .../09-libunbound8_1.13.1-1_amd64.deb ...
Unpacking libunbound8:amd64 (1.13.1-1) ...
Selecting previously unselected package libgnutls-dane0:amd64.
Preparing to unpack .../10-libgnutls-dane0_3.7.1-5+deb11u2_amd64.deb ...
Unpacking libgnutls-dane0:amd64 (3.7.1-5+deb11u2) ...
Selecting previously unselected package libidn11:amd64.
Preparing to unpack .../11-libidn11_1.33-3_amd64.deb ...
Unpacking libidn11:amd64 (1.33-3) ...
Selecting previously unselected package exim4-daemon-light.
Preparing to unpack .../12-exim4-daemon-light_4.94.2-7_amd64.deb ...
Unpacking exim4-daemon-light (4.94.2-7) ...
Selecting previously unselected package gsasl-common.
Preparing to unpack .../13-gsasl-common_1.10.0-4+deb11u1_all.deb ...
Unpacking gsasl-common (1.10.0-4+deb11u1) ...
Selecting previously unselected package libgc1:amd64.
Preparing to unpack .../14-libgc1_1%3a8.0.4-3_amd64.deb ...
Unpacking libgc1:amd64 (1:8.0.4-3) ...
Selecting previously unselected package libltdl7:amd64.
Preparing to unpack .../15-libltdl7_2.4.6-15_amd64.deb ...
Unpacking libltdl7:amd64 (2.4.6-15) ...
Selecting previously unselected package guile-2.2-libs:amd64.
Preparing to unpack .../16-guile-2.2-libs_2.2.7+1-6_amd64.deb ...
Unpacking guile-2.2-libs:amd64 (2.2.7+1-6) ...
Selecting previously unselected package libexpat1:amd64.
Preparing to unpack .../17-libexpat1_2.2.10-2+deb11u5_amd64.deb ...
Unpacking libexpat1:amd64 (2.2.10-2+deb11u5) ...
Selecting previously unselected package libfribidi0:amd64.
Preparing to unpack .../18-libfribidi0_1.0.8-2+deb11u1_amd64.deb ...
Unpacking libfribidi0:amd64 (1.0.8-2+deb11u1) ...
Selecting previously unselected package libntlm0:amd64.
Preparing to unpack .../19-libntlm0_1.6-3_amd64.deb ...
Unpacking libntlm0:amd64 (1.6-3) ...
Selecting previously unselected package libgsasl7:amd64.
Preparing to unpack .../20-libgsasl7_1.10.0-4+deb11u1_amd64.deb ...
Unpacking libgsasl7:amd64 (1.10.0-4+deb11u1) ...
Selecting previously unselected package libsasl2-modules-db:amd64.
Preparing to unpack .../21-libsasl2-modules-db_2.1.27+dfsg-2.1+deb11u1_amd64.deb ...
Unpacking libsasl2-modules-db:amd64 (2.1.27+dfsg-2.1+deb11u1) ...
Selecting previously unselected package libsasl2-2:amd64.
Preparing to unpack .../22-libsasl2-2_2.1.27+dfsg-2.1+deb11u1_amd64.deb ...
Unpacking libsasl2-2:amd64 (2.1.27+dfsg-2.1+deb11u1) ...
Selecting previously unselected package libldap-2.4-2:amd64.
Preparing to unpack .../23-libldap-2.4-2_2.4.57+dfsg-3+deb11u1_amd64.deb ...
Unpacking libldap-2.4-2:amd64 (2.4.57+dfsg-3+deb11u1) ...
Selecting previously unselected package libldap-common.
Preparing to unpack .../24-libldap-common_2.4.57+dfsg-3+deb11u1_all.deb ...
Unpacking libldap-common (2.4.57+dfsg-3+deb11u1) ...
Selecting previously unselected package mailutils-common.
Preparing to unpack .../25-mailutils-common_1%3a3.10-3_all.deb ...
Unpacking mailutils-common (1:3.10-3) ...
Selecting previously unselected package mysql-common.
Preparing to unpack .../26-mysql-common_5.8+1.0.7_all.deb ...
Unpacking mysql-common (5.8+1.0.7) ...
Selecting previously unselected package mariadb-common.
Preparing to unpack .../27-mariadb-common_1%3a10.5.18-0+deb11u1_all.deb ...
Unpacking mariadb-common (1:10.5.18-0+deb11u1) ...
Selecting previously unselected package libmariadb3:amd64.
Preparing to unpack .../28-libmariadb3_1%3a10.5.18-0+deb11u1_amd64.deb ...
Unpacking libmariadb3:amd64 (1:10.5.18-0+deb11u1) ...
Selecting previously unselected package libpython3.9-minimal:amd64.
Preparing to unpack .../29-libpython3.9-minimal_3.9.2-1_amd64.deb ...
Unpacking libpython3.9-minimal:amd64 (3.9.2-1) ...
Selecting previously unselected package libmpdec3:amd64.
Preparing to unpack .../30-libmpdec3_2.5.1-1_amd64.deb ...
Unpacking libmpdec3:amd64 (2.5.1-1) ...
Selecting previously unselected package libsqlite3-0:amd64.
Preparing to unpack .../31-libsqlite3-0_3.34.1-3_amd64.deb ...
Unpacking libsqlite3-0:amd64 (3.34.1-3) ...
Selecting previously unselected package libpython3.9-stdlib:amd64.
Preparing to unpack .../32-libpython3.9-stdlib_3.9.2-1_amd64.deb ...
Unpacking libpython3.9-stdlib:amd64 (3.9.2-1) ...
Selecting previously unselected package libpython3.9:amd64.
Preparing to unpack .../33-libpython3.9_3.9.2-1_amd64.deb ...
Unpacking libpython3.9:amd64 (3.9.2-1) ...
Selecting previously unselected package libmailutils7:amd64.
Preparing to unpack .../34-libmailutils7_1%3a3.10-3+b1_amd64.deb ...
Unpacking libmailutils7:amd64 (1:3.10-3+b1) ...
Selecting previously unselected package libsasl2-modules:amd64.
Preparing to unpack .../35-libsasl2-modules_2.1.27+dfsg-2.1+deb11u1_amd64.deb ...
Unpacking libsasl2-modules:amd64 (2.1.27+dfsg-2.1+deb11u1) ...
Selecting previously unselected package mailutils.
Preparing to unpack .../36-mailutils_1%3a3.10-3+b1_amd64.deb ...
Unpacking mailutils (1:3.10-3+b1) ...
Selecting previously unselected package psmisc.
Preparing to unpack .../37-psmisc_23.4-2_amd64.deb ...
Unpacking psmisc (23.4-2) ...
Setting up libexpat1:amd64 (2.2.10-2+deb11u5) ...
Setting up media-types (4.0.0) ...
Setting up mysql-common (5.8+1.0.7) ...
update-alternatives: using /etc/mysql/my.cnf.fallback to provide /etc/mysql/my.cnf (my.cnf) in auto mode
Setting up psmisc (23.4-2) ...
Setting up libpython3.9-minimal:amd64 (3.9.2-1) ...
Setting up libsqlite3-0:amd64 (3.34.1-3) ...
Setting up cron (3.0pl1-137) ...
Adding group `crontab' (GID 106) ...
Done.
Created symlink /etc/systemd/system/multi-user.target.wants/cron.service → /lib/systemd/system/cron.service.
Setting up libsasl2-modules:amd64 (2.1.27+dfsg-2.1+deb11u1) ...
Setting up libldap-common (2.4.57+dfsg-3+deb11u1) ...
Setting up libsasl2-modules-db:amd64 (2.1.27+dfsg-2.1+deb11u1) ...
Setting up mariadb-common (1:10.5.18-0+deb11u1) ...
update-alternatives: using /etc/mysql/mariadb.cnf to provide /etc/mysql/my.cnf (my.cnf) in auto mode
Setting up libntlm0:amd64 (1.6-3) ...
Setting up libidn11:amd64 (1.33-3) ...
Setting up libfribidi0:amd64 (1.0.8-2+deb11u1) ...
Setting up libevent-2.1-7:amd64 (2.1.12-stable-1) ...
Setting up mailutils-common (1:3.10-3) ...
Setting up libmariadb3:amd64 (1:10.5.18-0+deb11u1) ...
Setting up libgc1:amd64 (1:8.0.4-3) ...
Setting up libltdl7:amd64 (2.4.6-15) ...
Setting up libsasl2-2:amd64 (2.1.27+dfsg-2.1+deb11u1) ...
Setting up exim4-config (4.94.2-7) ...
Adding system-user for exim (v4)
Setting up libmpdec3:amd64 (2.5.1-1) ...
Setting up gsasl-common (1.10.0-4+deb11u1) ...
Setting up openssl (1.1.1n-0+deb11u3) ...
Setting up readline-common (8.1-1) ...
Setting up exim4-base (4.94.2-7) ...
exim: DB upgrade, deleting hints-db
Created symlink /etc/systemd/system/timers.target.wants/exim4-base.timer → /lib/systemd/system/exim4-base.timer.
exim4-base.service is a disabled or a static unit, not starting it.
Setting up libreadline8:amd64 (8.1-1) ...
Setting up libgsasl7:amd64 (1.10.0-4+deb11u1) ...
Setting up libldap-2.4-2:amd64 (2.4.57+dfsg-3+deb11u1) ...
Setting up ca-certificates (20210119) ...
Updating certificates in /etc/ssl/certs...
129 added, 0 removed; done.
Setting up libunbound8:amd64 (1.13.1-1) ...
Setting up guile-2.2-libs:amd64 (2.2.7+1-6) ...
Setting up libpython3.9-stdlib:amd64 (3.9.2-1) ...
Setting up libgnutls-dane0:amd64 (3.7.1-5+deb11u2) ...
Setting up libpython3.9:amd64 (3.9.2-1) ...
Setting up exim4-daemon-light (4.94.2-7) ...
Setting up libmailutils7:amd64 (1:3.10-3+b1) ...
Setting up mailutils (1:3.10-3+b1) ...
update-alternatives: using /usr/bin/frm.mailutils to provide /usr/bin/frm (frm) in auto mode
update-alternatives: using /usr/bin/from.mailutils to provide /usr/bin/from (from) in auto mode
update-alternatives: using /usr/bin/messages.mailutils to provide /usr/bin/messages (messages) in auto mode
update-alternatives: using /usr/bin/movemail.mailutils to provide /usr/bin/movemail (movemail) in auto mode
update-alternatives: using /usr/bin/readmsg.mailutils to provide /usr/bin/readmsg (readmsg) in auto mode
update-alternatives: using /usr/bin/dotlock.mailutils to provide /usr/bin/dotlock (dotlock) in auto mode
update-alternatives: using /usr/bin/mail.mailutils to provide /usr/bin/mailx (mailx) in auto mode
Processing triggers for libc-bin (2.31-13+deb11u5) ...
Processing triggers for ca-certificates (20210119) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
root@node2:~# crontab -e
no crontab for root - using an empty one
update-alternatives: error: no alternatives for editor
/usr/bin/sensible-editor: 25: editor: not found
/usr/bin/sensible-editor: 28: nano: not found
/usr/bin/sensible-editor: 31: nano-tiny: not found
/usr/bin/sensible-editor: 34: vi: not found
Couldn't find an editor!
Set the $EDITOR environment variable to your desired editor.
crontab: "/usr/bin/sensible-editor" exited with status 1
root@node2:~# apt-get install nano -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Suggested packages:
  hunspell
The following NEW packages will be installed:
  nano
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 657 kB of archives.
After this operation, 2,591 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian stable/main amd64 nano amd64 5.4-2+deb11u2 [657 kB]
Fetched 657 kB in 0s (2,847 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package nano.
(Reading database ... 11452 files and directories currently installed.)
Preparing to unpack .../nano_5.4-2+deb11u2_amd64.deb ...
Unpacking nano (5.4-2+deb11u2) ...
Setting up nano (5.4-2+deb11u2) ...
update-alternatives: using /bin/nano to provide /usr/bin/editor (editor) in auto mode
update-alternatives: using /bin/nano to provide /usr/bin/pico (pico) in auto mode
root@node2:~# echo "* */60 * * * root 'rsync -a /var/log/apt/ --exclude="term.log" node1:/root/backup_logs/'" >> /etc/crontab
root@node2:~# ssh node1
Linux node1 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb  2 12:03:22 2023 from 10.0.3.1
root@node1:~# ls -la backup_logs/
total 20
drwxr-xr-x 2 root root 4096 Feb  2 11:48 .
drwxr-xr-x 3 root root 4096 Feb  2 11:49 ..
-rw-r--r-- 1 root root 6200 Feb  2 15:12 eipp.log.xz
-rw-r--r-- 1 root root  178 Feb  2 15:12 history.log
```
# Setup systemd timer on node2 only! 
```sh
root@node2:~# systemd status *timer
Excess arguments.
root@node2:~# systemctl status *timer
● apt-daily.timer - Daily apt download activities
     Loaded: loaded (/lib/systemd/system/apt-daily.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Thu 2023-02-02 11:35:49 UTC; 36min ago
    Trigger: Fri 2023-02-03 01:35:49 UTC; 13h left
   Triggers: ● apt-daily.service

Feb 02 11:35:49 node2 systemd[1]: Started Daily apt download activities.

● e2scrub_all.timer - Periodic ext4 Online Metadata Check for All Filesystems
     Loaded: loaded (/lib/systemd/system/e2scrub_all.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Thu 2023-02-02 11:35:49 UTC; 36min ago
    Trigger: Sun 2023-02-05 03:10:56 UTC; 2 days left
   Triggers: ● e2scrub_all.service

Feb 02 11:35:49 node2 systemd[1]: Started Periodic ext4 Online Metadata Check for All Filesystems.

● apt-daily-upgrade.timer - Daily apt upgrade and clean activities
     Loaded: loaded (/lib/systemd/system/apt-daily-upgrade.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Thu 2023-02-02 11:35:49 UTC; 36min ago
    Trigger: Fri 2023-02-03 06:10:56 UTC; 17h left
   Triggers: ● apt-daily-upgrade.service

Feb 02 11:35:49 node2 systemd[1]: Started Daily apt upgrade and clean activities.

● systemd-tmpfiles-clean.timer - Daily Cleanup of Temporary Directories
     Loaded: loaded (/lib/systemd/system/systemd-tmpfiles-clean.timer; static)
     Active: active (waiting) since Thu 2023-02-02 11:35:49 UTC; 36min ago
    Trigger: Fri 2023-02-03 11:50:50 UTC; 23h left
   Triggers: ● systemd-tmpfiles-clean.service
       Docs: man:tmpfiles.d(5)
             man:systemd-tmpfiles(8)

Feb 02 11:35:49 node2 systemd[1]: Started Daily Cleanup of Temporary Directories.

● exim4-base.timer - Daily exim4-base housekeeping
     Loaded: loaded (/lib/systemd/system/exim4-base.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Thu 2023-02-02 11:52:56 UTC; 19min ago
    Trigger: Fri 2023-02-03 00:00:00 UTC; 11h left
   Triggers: ● exim4-base.service
       Docs: man:exim4(8)

Feb 02 11:52:56 node2 systemd[1]: Started Daily exim4-base housekeeping.
root@node2:~# cd /etc/systemd/system
root@node2:/etc/systemd/system# ls
default.target        getty-static.service  multi-user.target.wants      sshd.service          systemd-udevd.service  udev.service
default.target.wants  getty.target.wants    network-online.target.wants  sysinit.target.wants  timers.target.wants
root@node2:/etc/systemd/system# cat sshd.service
[Unit]
Description=OpenBSD Secure Shell server
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target auditd.service
ConditionPathExists=!/etc/ssh/sshd_not_to_be_run

[Service]
EnvironmentFile=-/etc/default/ssh
ExecStartPre=/usr/sbin/sshd -t
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
ExecReload=/usr/sbin/sshd -t
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartPreventExitStatus=255
Type=notify
RuntimeDirectory=sshd
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
Alias=sshd.service
root@node2:/etc/systemd/system# cat <<EOF > rsyng-apt-log.service
> [UNIT]
> Description=rsync from node2:/var/log/apt/ to node1:/root/backup_logs/
> After=network.target
>
> [Install]
> WantedBy=multi-user.target
>
> [Service]
> Type=oneshot
> ExecStart=/usr/bin/rsync -a /var/log/apt/ --exclude='term.log' node1:/root/backup_logs/
> EOF
root@node2:/etc/systemd/system# cat rsyng-apt-log.service
[Unit]
Description=rsync from node2:/var/log/apt/ to node1:/root/backup_logs/
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
Type=oneshot
ExecStart=/usr/bin/rsync -a /var/log/apt/ --exclude='term.log' node1:/root/backup_logs/
root@node2:/etc/systemd/system# systemctl start rsyng-apt-log.service
root@node2:/etc/systemd/system# ssh node1
Linux node1 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb  2 12:08:40 2023 from 10.0.3.141
root@node1:~# ls -la backup_logs/
total 20
drwxr-xr-x 2 root root 4096 Feb  2 11:54 .
drwx------ 4 root root 4096 Feb  2 11:47 ..
-rw-r--r-- 1 root root 7780 Feb  2 11:54 eipp.log.xz
-rw-r--r-- 1 root root 2146 Feb  2 11:54 history.log

```
