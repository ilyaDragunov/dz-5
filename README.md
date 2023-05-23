  Домашнее задание 5

Цель домашнего задания:

- Научиться самостоятельно развернуть сервис NFS и подключить к нему клиента

Описание домашнего задания:

Основная часть:
- `vagrant up` должен поднимать 2 настроенных виртуальных машины (сервер NFS и клиента) без дополнительных ручн>
- в экспортированной директории должна быть поддиректория с именем __upload__ с правами на запись в неё;
- экспортированная директория должна автоматически монтироваться на клиенте при старте виртуальной машины (syst>
- монтирование и работа NFS на клиенте должна быть организована с использованием NFSv3 по протоколу UDP;
- firewall должен быть включен и настроен как на клиенте, так и на сервере.
Для самостоятельной реализации:
- настроить аутентификацию через KERBEROS с использованием NFSv4. ##


                Выполнение домашнего задания

1. Создаём тестовые виртуальные машины

```

#ruby
# -*- mode: ruby -*-
# vi: set ft=ruby : vsa
Vagrant.configure(2) do |config|
 config.vm.synced_folder '.', '/vagrant', disabled: true
 config.vm.box = "centos/7"
 config.vm.box_version = "2004.01"
 config.vm.provider "virtualbox" do |v|
 v.memory = 256
 v.cpus = 1
 end
 config.vm.define "nfss" do |nfss|
 nfss.vm.network "private_network", ip: "192.168.56.101",  virtualbox__intnet: "net1"
 nfss.vm.hostname = "nfss"
 end
 config.vm.define "nfsc" do |nfsc|
 nfsc.vm.network "private_network", ip: "192.168.56.11",  virtualbox__intnet: "net1"
 nfsc.vm.hostname = "nfsc"
 end
end

```

```

user@ubuntu-vm:~/DZ-OTUS/DZ-5$ vagrant up
Bringing machine 'nfss' up with 'virtualbox' provider...
Bringing machine 'nfsc' up with 'virtualbox' provider...
==> nfss: Checking if box 'centos/7' version '2004.01' is up to date...
==> nfss: Clearing any previously set network interfaces...
==> nfss: Preparing network interfaces based on configuration...
    nfss: Adapter 1: nat
    nfss: Adapter 2: intnet
==> nfss: Forwarding ports...
    nfss: 22 (guest) => 2222 (host) (adapter 1)
==> nfss: Running 'pre-boot' VM customizations...
==> nfss: Booting VM...
==> nfss: Waiting for machine to boot. This may take a few minutes...
    nfss: SSH address: 127.0.0.1:2222
    nfss: SSH username: vagrant
    nfss: SSH auth method: private key
    nfss:
    nfss: Vagrant insecure key detected. Vagrant will automatically replace
    nfss: this with a newly generated keypair for better security.
    nfss:
    nfss: Inserting generated public key within guest...
    nfss: Removing insecure key from the guest if it's present...
    nfss: Key inserted! Disconnecting and reconnecting using new SSH key...
==> nfss: Machine booted and ready!
==> nfss: Checking for guest additions in VM...
    nfss: No guest additions were detected on the base box for this VM! Guest
    nfss: additions are required for forwarded ports, shared folders, host only
    nfss: networking, and more. If SSH fails on this machine, please install
    nfss: the guest additions and repackage the box to continue.
    nfss:
    nfss: This is not an error message; everything may continue to work properly,
    nfss: in which case you may ignore this message.
==> nfss: Setting hostname...
==> nfss: Configuring and enabling network interfaces...
==> nfsc: Importing base box 'centos/7'...
==> nfsc: Matching MAC address for NAT networking...
==> nfsc: Checking if box 'centos/7' version '2004.01' is up to date...
==> nfsc: Setting the name of the VM: DZ-5_nfsc_1684830661538_37349
==> nfsc: Fixed port collision for 22 => 2222. Now on port 2200.
==> nfsc: Clearing any previously set network interfaces...
==> nfsc: Preparing network interfaces based on configuration...
    nfsc: Adapter 1: nat
    nfsc: Adapter 2: intnet
==> nfsc: Forwarding ports...
    nfsc: 22 (guest) => 2200 (host) (adapter 1)
==> nfsc: Running 'pre-boot' VM customizations...
==> nfsc: Booting VM...
==> nfsc: Waiting for machine to boot. This may take a few minutes...
    nfsc: SSH address: 127.0.0.1:2200
    nfsc: SSH username: vagrant
    nfsc: SSH auth method: private key
    nfsc:
    nfsc: Vagrant insecure key detected. Vagrant will automatically replace
    nfsc: this with a newly generated keypair for better security.
    nfsc:
    nfsc: Inserting generated public key within guest...
    nfsc: Removing insecure key from the guest if it's present...
    nfsc: Key inserted! Disconnecting and reconnecting using new SSH key...
==> nfsc: Machine booted and ready!
==> nfsc: Checking for guest additions in VM...
    nfsc: No guest additions were detected on the base box for this VM! Guest
    nfsc: additions are required for forwarded ports, shared folders, host only
    nfsc: networking, and more. If SSH fails on this machine, please install
    nfsc: the guest additions and repackage the box to continue.
    nfsc:
    nfsc: This is not an error message; everything may continue to work properly,
    nfsc: in which case you may ignore this message.
==> nfsc: Setting hostname...
==> nfsc: Configuring and enabling network interfaces...

```

2. Настраиваем сервер NFS, заходим на сервер 

```

user@ubuntu-vm:~/DZ-OTUS/DZ-5$ bash vagrant ssh nfss
[vagrant@nfss ~]$

```

3. Cервер NFS уже установлен в CentOS 7 как часть дистрибутива, доставим утилиты ,которые облегчат отладку

```

[vagrant@nfss ~]$ sudo -i
[root@nfss ~]#
[root@nfss ~]# bash
[root@nfss ~]# yum install nfs-utils
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: ftp.plusline.net
 * extras: mirror.rackspeed.de
 * updates: mirror.rackspeed.de
base                                                                              | 3.6 kB  00:00:00
extras                                                                            | 2.9 kB  00:00:00
updates                                                                           | 2.9 kB  00:00:00
(1/4): base/7/x86_64/group_gz                                                     | 153 kB  00:00:00
(2/4): extras/7/x86_64/primary_db                                                 | 249 kB  00:00:00
(3/4): base/7/x86_64/primary_db                                                   | 6.1 MB  00:00:05
(4/4): updates/7/x86_64/primary_db                                                |  21 MB  00:00:10
Resolving Dependencies
--> Running transaction check
---> Package nfs-utils.x86_64 1:1.3.0-0.66.el7 will be updated
---> Package nfs-utils.x86_64 1:1.3.0-0.68.el7.2 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================
 Package                Arch                Version                           Repository            Size
=========================================================================================================
Updating:
 nfs-utils              x86_64              1:1.3.0-0.68.el7.2                updates              413 k

Transaction Summary
=========================================================================================================
Upgrade  1 Package

Total download size: 413 k
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for updates
warning: /var/cache/yum/x86_64/7/updates/packages/nfs-utils-1.3.0-0.68.el7.2.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for nfs-utils-1.3.0-0.68.el7.2.x86_64.rpm is not installed
nfs-utils-1.3.0-0.68.el7.2.x86_64.rpm                                             | 413 kB  00:00:00
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-8.2003.0.el7.centos.x86_64 (@anaconda)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : 1:nfs-utils-1.3.0-0.68.el7.2.x86_64                                                   1/2
  Cleanup    : 1:nfs-utils-1.3.0-0.66.el7.x86_64                                                     2/2
  Verifying  : 1:nfs-utils-1.3.0-0.68.el7.2.x86_64                                                   1/2
  Verifying  : 1:nfs-utils-1.3.0-0.66.el7.x86_64                                                     2/2

Updated:
  nfs-utils.x86_64 1:1.3.0-0.68.el7.2

Complete!

```

4. Включаем firewalL, либо же проверяем, что он работает

```

[root@nfss ~]# systemctl enable firewalld --now
Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.

[root@nfss ~]# iptables-save
# Generated by iptables-save v1.4.21 on Tue May 23 09:19:34 2023
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [1:76]
:POSTROUTING ACCEPT [1:76]
:OUTPUT_direct - [0:0]
:POSTROUTING_ZONES - [0:0]
:POSTROUTING_ZONES_SOURCE - [0:0]
:POSTROUTING_direct - [0:0]
:POST_public - [0:0]
:POST_public_allow - [0:0]
:POST_public_deny - [0:0]
:POST_public_log - [0:0]
:PREROUTING_ZONES - [0:0]
:PREROUTING_ZONES_SOURCE - [0:0]
:PREROUTING_direct - [0:0]
:PRE_public - [0:0]
:PRE_public_allow - [0:0]
:PRE_public_deny - [0:0]
:PRE_public_log - [0:0]
-A PREROUTING -j PREROUTING_direct
-A PREROUTING -j PREROUTING_ZONES_SOURCE
-A PREROUTING -j PREROUTING_ZONES
-A OUTPUT -j OUTPUT_direct
-A POSTROUTING -j POSTROUTING_direct
-A POSTROUTING -j POSTROUTING_ZONES_SOURCE
-A POSTROUTING -j POSTROUTING_ZONES
-A POSTROUTING_ZONES -o eth1 -g POST_public
-A POSTROUTING_ZONES -o eth0 -g POST_public
-A POSTROUTING_ZONES -g POST_public
-A POST_public -j POST_public_log
-A POST_public -j POST_public_deny
-A POST_public -j POST_public_allow
-A PREROUTING_ZONES -i eth1 -g PRE_public
-A PREROUTING_ZONES -i eth0 -g PRE_public
-A PREROUTING_ZONES -g PRE_public
-A PRE_public -j PRE_public_log
-A PRE_public -j PRE_public_deny
-A PRE_public -j PRE_public_allow
COMMIT
# Completed on Tue May 23 09:19:34 2023
# Generated by iptables-save v1.4.21 on Tue May 23 09:19:34 2023
*mangle
:PREROUTING ACCEPT [69:4000]
:INPUT ACCEPT [69:4000]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [36:3272]
:POSTROUTING ACCEPT [36:3272]
:FORWARD_direct - [0:0]
:INPUT_direct - [0:0]
:OUTPUT_direct - [0:0]
:POSTROUTING_direct - [0:0]
:PREROUTING_ZONES - [0:0]
:PREROUTING_ZONES_SOURCE - [0:0]
:PREROUTING_direct - [0:0]
:PRE_public - [0:0]
:PRE_public_allow - [0:0]
:PRE_public_deny - [0:0]
:PRE_public_log - [0:0]
-A PREROUTING -j PREROUTING_direct
-A PREROUTING -j PREROUTING_ZONES_SOURCE
-A PREROUTING -j PREROUTING_ZONES
-A INPUT -j INPUT_direct
-A FORWARD -j FORWARD_direct
-A OUTPUT -j OUTPUT_direct
-A POSTROUTING -j POSTROUTING_direct
-A PREROUTING_ZONES -i eth1 -g PRE_public
-A PREROUTING_ZONES -i eth0 -g PRE_public
-A PREROUTING_ZONES -g PRE_public
-A PRE_public -j PRE_public_log
-A PRE_public -j PRE_public_deny
-A PRE_public -j PRE_public_allow
COMMIT
# Completed on Tue May 23 09:19:34 2023
# Generated by iptables-save v1.4.21 on Tue May 23 09:19:34 2023
*security
:INPUT ACCEPT [71:4116]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [37:3348]
:FORWARD_direct - [0:0]
:INPUT_direct - [0:0]
:OUTPUT_direct - [0:0]
-A INPUT -j INPUT_direct
-A FORWARD -j FORWARD_direct
-A OUTPUT -j OUTPUT_direct
COMMIT
# Completed on Tue May 23 09:19:34 2023
# Generated by iptables-save v1.4.21 on Tue May 23 09:19:34 2023
*raw
:PREROUTING ACCEPT [69:4000]
:OUTPUT ACCEPT [36:3272]
:OUTPUT_direct - [0:0]
:PREROUTING_ZONES - [0:0]
:PREROUTING_ZONES_SOURCE - [0:0]
:PREROUTING_direct - [0:0]
:PRE_public - [0:0]
:PRE_public_allow - [0:0]
:PRE_public_deny - [0:0]
:PRE_public_log - [0:0]
-A PREROUTING -j PREROUTING_direct
-A PREROUTING -j PREROUTING_ZONES_SOURCE
-A PREROUTING -j PREROUTING_ZONES
-A OUTPUT -j OUTPUT_direct
-A PREROUTING_ZONES -i eth1 -g PRE_public
-A PREROUTING_ZONES -i eth0 -g PRE_public
-A PREROUTING_ZONES -g PRE_public
-A PRE_public -j PRE_public_log
-A PRE_public -j PRE_public_deny
-A PRE_public -j PRE_public_allow
COMMIT
# Completed on Tue May 23 09:19:34 2023
# Generated by iptables-save v1.4.21 on Tue May 23 09:19:34 2023
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [36:3272]
:FORWARD_IN_ZONES - [0:0]
:FORWARD_IN_ZONES_SOURCE - [0:0]
:FORWARD_OUT_ZONES - [0:0]
:FORWARD_OUT_ZONES_SOURCE - [0:0]
:FORWARD_direct - [0:0]
:FWDI_public - [0:0]
:FWDI_public_allow - [0:0]
:FWDI_public_deny - [0:0]
:FWDI_public_log - [0:0]
:FWDO_public - [0:0]
:FWDO_public_allow - [0:0]
:FWDO_public_deny - [0:0]
:FWDO_public_log - [0:0]
:INPUT_ZONES - [0:0]
:INPUT_ZONES_SOURCE - [0:0]
:INPUT_direct - [0:0]
:IN_public - [0:0]
:IN_public_allow - [0:0]
:IN_public_deny - [0:0]
:IN_public_log - [0:0]
:OUTPUT_direct - [0:0]
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -j INPUT_direct
-A INPUT -j INPUT_ZONES_SOURCE
-A INPUT -j INPUT_ZONES
-A INPUT -m conntrack --ctstate INVALID -j DROP
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -i lo -j ACCEPT
-A FORWARD -j FORWARD_direct
-A FORWARD -j FORWARD_IN_ZONES_SOURCE
-A FORWARD -j FORWARD_IN_ZONES
-A FORWARD -j FORWARD_OUT_ZONES_SOURCE
-A FORWARD -j FORWARD_OUT_ZONES
-A FORWARD -m conntrack --ctstate INVALID -j DROP
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
-A OUTPUT -o lo -j ACCEPT
-A OUTPUT -j OUTPUT_direct
-A FORWARD_IN_ZONES -i eth1 -g FWDI_public
-A FORWARD_IN_ZONES -i eth0 -g FWDI_public
-A FORWARD_IN_ZONES -g FWDI_public
-A FORWARD_OUT_ZONES -o eth1 -g FWDO_public
-A FORWARD_OUT_ZONES -o eth0 -g FWDO_public
-A FORWARD_OUT_ZONES -g FWDO_public
-A FWDI_public -j FWDI_public_log
-A FWDI_public -j FWDI_public_deny
-A FWDI_public -j FWDI_public_allow
-A FWDI_public -p icmp -j ACCEPT
-A FWDO_public -j FWDO_public_log
-A FWDO_public -j FWDO_public_deny
-A FWDO_public -j FWDO_public_allow
-A INPUT_ZONES -i eth1 -g IN_public
-A INPUT_ZONES -i eth0 -g IN_public
-A INPUT_ZONES -g IN_public
-A IN_public -j IN_public_log
-A IN_public -j IN_public_deny
-A IN_public -j IN_public_allow
-A IN_public -p icmp -j ACCEPT
-A IN_public_allow -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW,UNTRACKED -j ACCEPT
COMMIT

[root@nfss ~]# systemctl status firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2023-05-23 09:18:19 UTC; 1min 42s ago
     Docs: man:firewalld(1)
 Main PID: 21729 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─21729 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid

May 23 09:18:18 nfss systemd[1]: Starting firewalld - dynamic firewall daemon...
May 23 09:18:19 nfss systemd[1]: Started firewalld - dynamic firewall daemon.
May 23 09:18:19 nfss firewalld[21729]: WARNING: AllowZoneDrifting is enabled. This is considered a...now.
Hint: Some lines were ellipsized, use -l to show in full.

```

5. Разрешаем в firewall доступ к сервисам NFS

```

[root@nfss ~]# bash
[root@nfss ~]# firewall-cmd --add-service="nfs3" \
> --add-service="rpc-bind" \
> --add-service="mountd" \
> --permanent
success
[root@nfss ~]# firewall-cmd --reload
success

```

6. Включаем сервер NFS 

```

[root@nfss ~]# bash
[root@nfss ~]# systemctl enable nfs --now
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.


[root@nfss ~]# systemctl status nfs
● nfs-server.service - NFS server and services
   Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; vendor preset: disabled)
   Active: active (exited) since Tue 2023-05-23 09:25:43 UTC; 1min 26s ago
  Process: 21994 ExecStartPost=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssproxy ; fi (code=exited, status=0/SUCCESS)
  Process: 21977 ExecStart=/usr/sbin/rpc.nfsd $RPCNFSDARGS (code=exited, status=0/SUCCESS)
  Process: 21976 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
 Main PID: 21977 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/nfs-server.service

May 23 09:25:43 nfss systemd[1]: Starting NFS server and services...
May 23 09:25:43 nfss systemd[1]: Started NFS server and services.

```

7. Проверяем наличие слушаемых портов 2049/udp, 2049/tcp, 20048/udp,  20048/tcp, 111/udp, 111/tcp

```

[root@nfss ~]# ss -tnplu
Netid State      Recv-Q Send-Q     Local Address:Port                    Peer Address:Port
udp   UNCONN     0      0              127.0.0.1:323                                *:*                   users:(("chronyd",pid=412,fd=5))
udp   UNCONN     0      0                      *:68                                 *:*                   users:(("dhclient",pid=2444,fd=6))
udp   UNCONN     0      0                      *:20048                              *:*                   users:(("rpc.mountd",pid=21975,fd=7))
udp   UNCONN     0      0                      *:111                                *:*                   users:(("rpcbind",pid=340,fd=6))
udp   UNCONN     0      0                      *:35223                              *:*                   users:(("rpc.statd",pid=21969,fd=8))
udp   UNCONN     0      0                      *:929                                *:*                   users:(("rpcbind",pid=340,fd=7))
udp   UNCONN     0      0              127.0.0.1:945                                *:*                   users:(("rpc.statd",pid=21969,fd=5))
udp   UNCONN     0      0                      *:54999                              *:*
udp   UNCONN     0      0                      *:2049                               *:*
udp   UNCONN     0      0                   [::]:55071                           [::]:*
udp   UNCONN     0      0                  [::1]:323                             [::]:*                   users:(("chronyd",pid=412,fd=6))
udp   UNCONN     0      0                   [::]:20048                           [::]:*                   users:(("rpc.mountd",pid=21975,fd=9))
udp   UNCONN     0      0                   [::]:53868                           [::]:*                   users:(("rpc.statd",pid=21969,fd=10))
udp   UNCONN     0      0                   [::]:111                             [::]:*                   users:(("rpcbind",pid=340,fd=9))
udp   UNCONN     0      0                   [::]:929                             [::]:*                   users:(("rpcbind",pid=340,fd=10))
udp   UNCONN     0      0                   [::]:2049                            [::]:*
tcp   LISTEN     0      64                     *:2049                               *:*
tcp   LISTEN     0      128                    *:111                                *:*                   users:(("rpcbind",pid=340,fd=8))
tcp   LISTEN     0      128                    *:20048                              *:*                   users:(("rpc.mountd",pid=21975,fd=8))
tcp   LISTEN     0      128                    *:22                                 *:*                   users:(("sshd",pid=614,fd=3))
tcp   LISTEN     0      64                     *:44760                              *:*
tcp   LISTEN     0      128                    *:58104                              *:*                   users:(("rpc.statd",pid=21969,fd=9))
tcp   LISTEN     0      100            127.0.0.1:25                                 *:*                   users:(("master",pid=744,fd=13))
tcp   LISTEN     0      64                  [::]:41695                           [::]:*
tcp   LISTEN     0      64                  [::]:2049                            [::]:*
tcp   LISTEN     0      128                 [::]:38625                           [::]:*                   users:(("rpc.statd",pid=21969,fd=11))
tcp   LISTEN     0      128                 [::]:111                             [::]:*                   users:(("rpcbind",pid=340,fd=11))
tcp   LISTEN     0      128                 [::]:20048                           [::]:*                   users:(("rpc.mountd",pid=21975,fd=10))
tcp   LISTEN     0      128                 [::]:22                              [::]:*                   users:(("sshd",pid=614,fd=4))
tcp   LISTEN     0      100                [::1]:25                              [::]:*                   users:(("master",pid=744,fd=14))
[root@nfss ~]#

```

8. Создаём и настраиваем директорию, которая будет экспортирована в будущем

```

[root@nfss ~]# mkdir -p /srv/share/upload
[root@nfss ~]# chown -R nfsnobody:nfsnobody /srv/share
[root@nfss ~]# chmod 0777 /srv/share/upload

```

9. Cоздаём в файле __/etc/exports__ структуру, которая позволит экспортировать ранее созданную директорию 

```

[root@nfss ~]# bash
[root@nfss ~]# cat << EOF > /etc/exports
> /srv/share 192.168.56.11/32(rw,sync,root_squash)
> EOF

[root@nfss ~]# cat /etc/exports
/srv/share 192.168.56.11/32(rw,sync,root_squash)

```

10. Экспортируем ранее созданную директорию 

```

[root@nfss ~]# bash
[root@nfss ~]# exportfs -r

```

11. Проверяем экспортированную директорию следующей командой

```

[root@nfss ~]# exportfs -s
/srv/share  192.168.56.11/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
[root@nfss ~]#

[root@nfss ~]# who
vagrant  pts/0        May 23 09:10 (10.0.2.2)
[root@nfss ~]# exit
exit
[root@nfss ~]#
[root@nfss ~]#
[root@nfss ~]#
[root@nfss ~]#
[root@nfss ~]# ps -aux | grep bash
vagrant   2787  0.0  0.0  15796     0 pts/0    Ss   09:10   0:00 -bash
vagrant   2808  0.0  0.0  15800    40 pts/0    S    09:12   0:00 bash
root      2828  0.0  0.4  15800  1024 pts/0    S    09:13   0:00 -bash
root     22200  0.0  0.2  12500   688 pts/0    S+   09:44   0:00 grep --color=auto bash
[root@nfss ~]# exit
logout
[vagrant@nfss ~]$

```

12. Настраиваем клиент NFS,  заходим на сервер 

```
user@ubuntu-vm:~/DZ-OTUS/DZ-5$ vagrant ssh nfsc

```

13.  Доустановим вспомогательные утилиты

```

[vagrant@nfsc ~]$ sudo -i
[root@nfsc ~]#
[root@nfsc ~]# bash
[root@nfsc ~]# yum install nfs-utils
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirror.imt-systems.com
 * extras: mirror.imt-systems.com
 * updates: mirror1.hs-esslingen.de
base                                                                                    | 3.6 kB  00:00:00
extras                                                                                  | 2.9 kB  00:00:00
updates                                                                                 | 2.9 kB  00:00:00
(1/4): base/7/x86_64/group_gz                                                           | 153 kB  00:00:00
(2/4): extras/7/x86_64/primary_db                                                       | 249 kB  00:00:00
(3/4): base/7/x86_64/primary_db                                                         | 6.1 MB  00:00:04
(4/4): updates/7/x86_64/primary_db                                                      |  21 MB  00:00:04
Resolving Dependencies
--> Running transaction check
---> Package nfs-utils.x86_64 1:1.3.0-0.66.el7 will be updated
---> Package nfs-utils.x86_64 1:1.3.0-0.68.el7.2 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

===============================================================================================================
 Package                  Arch                  Version                           Repository              Size
===============================================================================================================
Updating:
 nfs-utils                x86_64                1:1.3.0-0.68.el7.2                updates                413 k

Transaction Summary
===============================================================================================================
Upgrade  1 Package

Total download size: 413 k
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for updates
warning: /var/cache/yum/x86_64/7/updates/packages/nfs-utils-1.3.0-0.68.el7.2.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for nfs-utils-1.3.0-0.68.el7.2.x86_64.rpm is not installed
nfs-utils-1.3.0-0.68.el7.2.x86_64.rpm                                                   | 413 kB  00:00:00
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-8.2003.0.el7.centos.x86_64 (@anaconda)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : 1:nfs-utils-1.3.0-0.68.el7.2.x86_64                                                         1/2
  Cleanup    : 1:nfs-utils-1.3.0-0.66.el7.x86_64                                                           2/2
  Verifying  : 1:nfs-utils-1.3.0-0.68.el7.2.x86_64                                                         1/2
  Verifying  : 1:nfs-utils-1.3.0-0.66.el7.x86_64                                                           2/2

Updated:
  nfs-utils.x86_64 1:1.3.0-0.68.el7.2

Complete!

```

14. Включаем firewalL, либо же проверяем, что он работает

```

[root@nfsc ~]#  systemctl enable firewalld --now
Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.


[root@nfsc ~]#
[root@nfsc ~]#
[root@nfsc ~]# iptables-save
# Generated by iptables-save v1.4.21 on Tue May 23 09:50:08 2023
*nat
:PREROUTING ACCEPT [1:84]
:INPUT ACCEPT [1:84]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
:OUTPUT_direct - [0:0]
:POSTROUTING_ZONES - [0:0]
:POSTROUTING_ZONES_SOURCE - [0:0]
:POSTROUTING_direct - [0:0]
:POST_public - [0:0]
:POST_public_allow - [0:0]
:POST_public_deny - [0:0]
:POST_public_log - [0:0]
:PREROUTING_ZONES - [0:0]
:PREROUTING_ZONES_SOURCE - [0:0]
:PREROUTING_direct - [0:0]
:PRE_public - [0:0]
:PRE_public_allow - [0:0]
:PRE_public_deny - [0:0]
:PRE_public_log - [0:0]
-A PREROUTING -j PREROUTING_direct
-A PREROUTING -j PREROUTING_ZONES_SOURCE
-A PREROUTING -j PREROUTING_ZONES
-A OUTPUT -j OUTPUT_direct
-A POSTROUTING -j POSTROUTING_direct
-A POSTROUTING -j POSTROUTING_ZONES_SOURCE
-A POSTROUTING -j POSTROUTING_ZONES
-A POSTROUTING_ZONES -o eth1 -g POST_public
-A POSTROUTING_ZONES -o eth0 -g POST_public
-A POSTROUTING_ZONES -g POST_public
-A POST_public -j POST_public_log
-A POST_public -j POST_public_deny
-A POST_public -j POST_public_allow
-A PREROUTING_ZONES -i eth1 -g PRE_public
-A PREROUTING_ZONES -i eth0 -g PRE_public
-A PREROUTING_ZONES -g PRE_public
-A PRE_public -j PRE_public_log
-A PRE_public -j PRE_public_deny
-A PRE_public -j PRE_public_allow
COMMIT
# Completed on Tue May 23 09:50:08 2023
# Generated by iptables-save v1.4.21 on Tue May 23 09:50:08 2023
*mangle
:PREROUTING ACCEPT [4:240]
:INPUT ACCEPT [4:240]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [2:160]
:POSTROUTING ACCEPT [2:160]
:FORWARD_direct - [0:0]
:INPUT_direct - [0:0]
:OUTPUT_direct - [0:0]
:POSTROUTING_direct - [0:0]
:PREROUTING_ZONES - [0:0]
:PREROUTING_ZONES_SOURCE - [0:0]
:PREROUTING_direct - [0:0]
:PRE_public - [0:0]
:PRE_public_allow - [0:0]
:PRE_public_deny - [0:0]
:PRE_public_log - [0:0]
-A PREROUTING -j PREROUTING_direct
-A PREROUTING -j PREROUTING_ZONES_SOURCE
-A PREROUTING -j PREROUTING_ZONES
-A INPUT -j INPUT_direct
-A FORWARD -j FORWARD_direct
-A OUTPUT -j OUTPUT_direct
-A POSTROUTING -j POSTROUTING_direct
-A PREROUTING_ZONES -i eth1 -g PRE_public
-A PREROUTING_ZONES -i eth0 -g PRE_public
-A PREROUTING_ZONES -g PRE_public
-A PRE_public -j PRE_public_log
-A PRE_public -j PRE_public_deny
-A PRE_public -j PRE_public_allow
COMMIT
# Completed on Tue May 23 09:50:08 2023
# Generated by iptables-save v1.4.21 on Tue May 23 09:50:08 2023
*security
:INPUT ACCEPT [4:240]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [2:160]
:FORWARD_direct - [0:0]
:INPUT_direct - [0:0]
:OUTPUT_direct - [0:0]
-A INPUT -j INPUT_direct
-A FORWARD -j FORWARD_direct
-A OUTPUT -j OUTPUT_direct
COMMIT
# Completed on Tue May 23 09:50:08 2023
# Generated by iptables-save v1.4.21 on Tue May 23 09:50:08 2023
*raw
:PREROUTING ACCEPT [4:240]
:OUTPUT ACCEPT [2:160]
:OUTPUT_direct - [0:0]
:PREROUTING_ZONES - [0:0]
:PREROUTING_ZONES_SOURCE - [0:0]
:PREROUTING_direct - [0:0]
:PRE_public - [0:0]
:PRE_public_allow - [0:0]
:PRE_public_deny - [0:0]
:PRE_public_log - [0:0]
-A PREROUTING -j PREROUTING_direct
-A PREROUTING -j PREROUTING_ZONES_SOURCE
-A PREROUTING -j PREROUTING_ZONES
-A OUTPUT -j OUTPUT_direct
-A PREROUTING_ZONES -i eth1 -g PRE_public
-A PREROUTING_ZONES -i eth0 -g PRE_public
-A PREROUTING_ZONES -g PRE_public
-A PRE_public -j PRE_public_log
-A PRE_public -j PRE_public_deny
-A PRE_public -j PRE_public_allow
COMMIT
# Completed on Tue May 23 09:50:08 2023
# Generated by iptables-save v1.4.21 on Tue May 23 09:50:08 2023
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [2:160]
:FORWARD_IN_ZONES - [0:0]
:FORWARD_IN_ZONES_SOURCE - [0:0]
:FORWARD_OUT_ZONES - [0:0]
:FORWARD_OUT_ZONES_SOURCE - [0:0]
:FORWARD_direct - [0:0]
:FWDI_public - [0:0]
:FWDI_public_allow - [0:0]
:FWDI_public_deny - [0:0]
:FWDI_public_log - [0:0]
:FWDO_public - [0:0]
:FWDO_public_allow - [0:0]
:FWDO_public_deny - [0:0]
:FWDO_public_log - [0:0]
:INPUT_ZONES - [0:0]
:INPUT_ZONES_SOURCE - [0:0]
:INPUT_direct - [0:0]
:IN_public - [0:0]
:IN_public_allow - [0:0]
:IN_public_deny - [0:0]
:IN_public_log - [0:0]
:OUTPUT_direct - [0:0]
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -j INPUT_direct
-A INPUT -j INPUT_ZONES_SOURCE
-A INPUT -j INPUT_ZONES
-A INPUT -m conntrack --ctstate INVALID -j DROP
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -i lo -j ACCEPT
-A FORWARD -j FORWARD_direct
-A FORWARD -j FORWARD_IN_ZONES_SOURCE
-A FORWARD -j FORWARD_IN_ZONES
-A FORWARD -j FORWARD_OUT_ZONES_SOURCE
-A FORWARD -j FORWARD_OUT_ZONES
-A FORWARD -m conntrack --ctstate INVALID -j DROP
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
-A OUTPUT -o lo -j ACCEPT
-A OUTPUT -j OUTPUT_direct
-A FORWARD_IN_ZONES -i eth1 -g FWDI_public
-A FORWARD_IN_ZONES -i eth0 -g FWDI_public
-A FORWARD_IN_ZONES -g FWDI_public
-A FORWARD_OUT_ZONES -o eth1 -g FWDO_public
-A FORWARD_OUT_ZONES -o eth0 -g FWDO_public
-A FORWARD_OUT_ZONES -g FWDO_public
-A FWDI_public -j FWDI_public_log
-A FWDI_public -j FWDI_public_deny
-A FWDI_public -j FWDI_public_allow
-A FWDI_public -p icmp -j ACCEPT
-A FWDO_public -j FWDO_public_log
-A FWDO_public -j FWDO_public_deny
-A FWDO_public -j FWDO_public_allow
-A INPUT_ZONES -i eth1 -g IN_public
-A INPUT_ZONES -i eth0 -g IN_public
-A INPUT_ZONES -g IN_public
-A IN_public -j IN_public_log
-A IN_public -j IN_public_deny
-A IN_public -j IN_public_allow
-A IN_public -p icmp -j ACCEPT
-A IN_public_allow -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW,UNTRACKED -j ACCEPT
COMMIT
# Completed on Tue May 23 09:50:08 2023
[root@nfsc ~]#
[root@nfsc ~]#  systemctl status firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2023-05-23 09:49:56 UTC; 32s ago
     Docs: man:firewalld(1)
 Main PID: 21703 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─21703 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid

May 23 09:49:55 nfsc systemd[1]: Starting firewalld - dynamic firewall daemon...
May 23 09:49:56 nfsc systemd[1]: Started firewalld - dynamic firewall daemon.
May 23 09:49:56 nfsc firewalld[21703]: WARNING: AllowZoneDrifting is enabled. This is considered an ins... now.
Hint: Some lines were ellipsized, use -l to show in full.

```

15.  Добавляем в __/etc/fstab__ строку_ echo "192.168.56.10:/srv/share/ /mnt nf
     s vers=3,proto=udp,noauto,x-systemd.automount 0 0" >> /etc/fstab

```

[root@nfsc ~]# echo "192.168.56.10:/srv/share/ /mnt nfs vers=3,proto=udp,noauto,x-systemd.automount 0 0" >> /etc/fstab
[root@nfsc ~]#
[root@nfsc ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Thu Apr 30 22:04:55 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=1c419d6c-5064-4a2b-953c-05b2c67edb15 /                       xfs     defaults        0 0
/swapfile none swap defaults 0 0
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
#VAGRANT-END
192.168.56.101:/srv/share/ /mnt nfs vers=3,proto=udp,noauto,x-systemd.automount 0 0

```

16. Перезапускаем службы daemon-reload и  remote-fs.target

```

[root@nfsc ~]# systemctl daemon-reload
[root@nfsc ~]# systemctl restart remote-fs.target

```

17. Отметим, что в данном случае происходит автоматическая генерация systemd units в каталоге
    `/run/systemd/generator/`, которые производят монтирование при первом обращении к катаmcлогу
    `/mnt/` - заходим в директорию `/mnt/` и проверяем успешность монтировани

```
[root@nfsc ~]# systemctl daemon-reload
[root@nfsc ~]# systemctl daemon-reload
[root@nfsc ~]# systemctl restart remote-fs.target
[root@nfsc ~]# systemctl status remote-fs.target
● remote-fs.target - Remote File Systems
   Loaded: loaded (/usr/lib/systemd/system/remote-fs.target; enabled; vendor preset: enabled)
   Active: active since Tue 2023-05-23 10:55:39 UTC; 19s ago
     Docs: man:systemd.special(7)

May 23 10:55:39 nfsc systemd[1]: Stopped target Remote File Systems.
May 23 10:55:39 nfsc systemd[1]: Stopping Remote File Systems.
May 23 10:55:39 nfsc systemd[1]: Reached target Remote File Systems.
[root@nfsc ~]# df -h
Filesystem                  Size  Used Avail Use% Mounted on
devtmpfs                    111M     0  111M   0% /dev
tmpfs                       118M     0  118M   0% /dev/shm
tmpfs                       118M  4.5M  114M   4% /run
tmpfs                       118M     0  118M   0% /sys/fs/cgroup
/dev/sda1                    40G  3.2G   37G   8% /
192.168.56.101:/srv/share/   40G  3.2G   37G   8% /mnt
tmpfs                        24M     0   24M   0% /run/user/1000

[root@nfsc upload]$ mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=21,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=10943)
192.168.56.101:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=32768,wsize=32768,namlen=255,hard,proto=udp,timeo=11,retrans=3,sec=sys,mountaddr=192.168.56.101,mountvers=3,mountport=20048,mountproto=udp,local_lock=none,addr=192.168.56.101)


```

18. Проверка работоспособности

- заходим на сервер 
- заходим в каталог `/srv/share/upload` 
- создаём тестовый файл `touch check_file` 
- заходим на клиент 
- заходим в каталог `/mnt/upload` 
- проверяем наличие ранее созданного файла 
- создаём тестовый файл `touch client_file` 
- проверяем, что файл успешно создан 

```

user@ubuntu-vm:~/DZ-OTUS/DZ-5$ vagrant ssh nfss
Last login: Tue May 23 10:51:06 2023 from 10.0.2.2

[vagrant@nfss ~]$ cd /srv/share/upload

[vagrant@nfss upload]$ touch check_file
[vagrant@nfss upload]$ exit
logout
user@ubuntu-vm:~/DZ-OTUS/DZ-5$ vagrant ssh nfsc
Last login: Tue May 23 10:46:19 2023 from 10.0.2.2

[vagrant@nfsc ~]$ cd /mnt/upload/
[vagrant@nfsc upload]$ ls
check_file

[vagrant@nfsc upload]$ touch touch client_file
[vagrant@nfsc upload]$ ls
check_file  client_file  touch
[vagrant@nfsc upload]$ exit
logout
user@ubuntu-vm:~/DZ-OTUS/DZ-5$ vagrant ssh nfss
Last login: Tue May 23 10:51:36 2023 from 10.0.2.2

[vagrant@nfss ~]$ cd /srv/share/upload/
[vagrant@nfss upload]$ ls
check_file  client_file  touch

```

19. Проверяем сервер.

- заходим на сервер в отдельном окне терминала 
- перезагружаем сервер 
- заходим на сервер 
- проверяем наличие файлов в каталоге `/srv/share/upload/` - проверяем статус сервера NFS `systemctl status nfs` - проверяем статус firewall `systemctl status firewalld` - проверяем экспорты `exportfs -s` 
- проверяем работу RPC `showmount -a 192.168.56.101` 

```

[vagrant@nfss ~]$ reboot
==== AUTHENTICATING FOR org.freedesktop.login1.reboot ===
Authentication is required for rebooting the system.
Authenticating as: root
Password:
==== AUTHENTICATION COMPLETE ===
Connection to 127.0.0.1 closed by remote host.

user@ubuntu-vm:~/DZ-OTUS/DZ-5$ vagrant ssh nfss
Last login: Tue May 23 11:04:01 2023 from 10.0.2.2

[vagrant@nfss ~]$ cd /srv/share/upload/

[vagrant@nfss upload]$ ls -llk
total 0
-rw-rw-r--. 1 vagrant vagrant 0 May 23 10:52 check_file
-rw-rw-r--. 1 vagrant vagrant 0 May 23 10:52 client_file
-rw-rw-r--. 1 vagrant vagrant 0 May 23 10:52 touch

[vagrant@nfss upload]$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2023-05-23 11:05:49 UTC; 17min ago
     Docs: man:firewalld(1)
 Main PID: 405 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─405 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid


 nfs-server.service - NFS server and services
   Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; vendor preset: disabled)
  Drop-In: /run/systemd/generator/nfs-server.service.d
           └─order-with-mounts.conf
   Active: active (exited) since Tue 2023-05-23 11:05:55 UTC; 17min ago
  Process: 818 ExecStartPost=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssproxy ; fi (code=exited, status=0/SUCCESS)
  Process: 792 ExecStart=/usr/sbin/rpc.nfsd $RPCNFSDARGS (code=exited, status=0/SUCCESS)
  Process: 788 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
 Main PID: 792 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/nfs-server.service


[vagrant@nfss upload]$ sudo -i
[root@nfss ~]# exportfs -s
/srv/share  192.168.56.11/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)

[root@nfss ~]# showmount -a 192.168.56.101
All mount points on 192.168.56.101:

```

20. Проверяем клиен

- возвращаемся на клиент 
- перезагружаем клиент 
- заходим на клиент 
- проверяем работу RPC `showmount -a 192.168.50.10` - заходим в каталог `/mnt/upload` 
- проверяем статус монтирования `mount | grep mnt` 
- проверяем наличие ранее созданных файлов 
- создаём тестовый файл `touch final_check` 
- проверяем, что файл успешно создан 


```

[vagrant@nfsc ~]$ reboot
==== AUTHENTICATING FOR org.freedesktop.login1.reboot ===
Authentication is required for rebooting the system.
Authenticating as: root
Password:
==== AUTHENTICATION COMPLETE ===
Connection to 127.0.0.1 closed by remote host.
user@ubuntu-vm:~/DZ-OTUS/DZ-5$


user@ubuntu-vm:~/DZ-OTUS/DZ-5$ vagrant ssh nfsc
Last login: Tue May 23 11:26:42 2023 from 10.0.2.2


[vagrant@nfsc ~]$ showmount -a 192.168.56.101
All mount points on 192.168.56.101:

[vagrant@nfsc ~]$ cd /mnt/upload/
[vagrant@nfsc upload]$ ls -llk
total 0
-rw-rw-r--. 1 vagrant vagrant 0 May 23 10:52 check_file
-rw-rw-r--. 1 vagrant vagrant 0 May 23 10:52 client_file
-rw-rw-r--. 1 vagrant vagrant 0 May 23 10:52 touch


[vagrant@nfsc upload]$ mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=21,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=10943)
192.168.56.101:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=32768,wsize=32768,namlen=255,hard,proto=udp,timeo=11,retrans=3,sec=sys,mountaddr=192.168.56.101,mountvers=3,mountport=20048,mountproto=udp,local_lock=none,addr=192.168.56.101)


[vagrant@nfsc upload]$ showmount -a 192.168.56.101
All mount points on 192.168.56.101:
192.168.56.11:/srv/share

[vagrant@nfsc upload]$ touch final_check /mnt/upload/

[vagrant@nfsc upload]$ ls -llk /mnt/upload/
total 0
-rw-rw-r--. 1 vagrant vagrant 0 May 23 10:52 check_file
-rw-rw-r--. 1 vagrant vagrant 0 May 23 10:52 client_file
-rw-rw-r--. 1 vagrant vagrant 0 May 23 11:33 final_check

# dz-5
