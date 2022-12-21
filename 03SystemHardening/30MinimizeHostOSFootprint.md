# マニュアル

# 学習ログ
## 不要なサービスの停止

停止するサービス（ここではsnapd）の停止
```
$ systemctl status snapd
● snapd.service - Snap Daemon
     Loaded: loaded (/lib/systemd/system/snapd.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-12-20 08:33:39 UTC; 4h 50min ago
TriggeredBy: ● snapd.socket
   Main PID: 801 (snapd)
      Tasks: 12 (limit: 9444)
     Memory: 292.0M
     CGroup: /system.slice/snapd.service
             mq801 /usr/lib/snapd/snapd

Dec 20 08:33:39 master-20221214 snapd[801]: overlord.go:268: Acquired state lock file
Dec 20 08:33:39 master-20221214 snapd[801]: daemon.go:247: started snapd/2.57.6 (series 16; classic) ubuntu/20.04 (amd64) linux/5.15.0-1025-oracle.
Dec 20 08:33:39 master-20221214 snapd[801]: daemon.go:340: adjusting startup timeout by 55s (pessimistic estimate of 30s plus 5s per snap)
Dec 20 08:33:39 master-20221214 systemd[1]: Started Snap Daemon.
Dec 20 08:33:41 master-20221214 snapd[801]: storehelpers.go:748: cannot refresh: snap has no updates available: "core20", "lxd", "snapd"
Dec 20 08:34:03 master-20221214 snapd[801]: services.go:1090: RemoveSnapServices - disabling snap.oracle-cloud-agent.oracle-cloud-agent.service
Dec 20 08:34:03 master-20221214 snapd[801]: services.go:1090: RemoveSnapServices - disabling snap.oracle-cloud-agent.oracle-cloud-agent-updater.service
Dec 20 08:34:08 master-20221214 snapd[801]: storehelpers.go:748: cannot refresh: snap has no updates available: "core18", "oracle-cloud-agent"
Dec 20 12:58:40 master-20221214 snapd[801]: storehelpers.go:748: cannot refresh: snap has no updates available: "core18", "core20", "lxd", "oracle-cloud-agent", "snapd"
Dec 20 12:58:40 master-20221214 snapd[801]: autorefresh.go:540: auto-refresh: all snaps are up-to-date
$ sudo systemctl stop snapd
Warning: Stopping snapd.service, but it can still be activated by:
  snapd.socket
$ systemctl status snapd
● snapd.service - Snap Daemon
     Loaded: loaded (/lib/systemd/system/snapd.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Tue 2022-12-20 13:25:17 UTC; 31s ago
TriggeredBy: ● snapd.socket
    Process: 801 ExecStart=/usr/lib/snapd/snapd (code=exited, status=0/SUCCESS)
   Main PID: 801 (code=exited, status=0/SUCCESS)

Dec 20 08:34:03 master-20221214 snapd[801]: services.go:1090: RemoveSnapServices - disabling snap.oracle-cloud-agent.oracle-cloud-agent.service
Dec 20 08:34:03 master-20221214 snapd[801]: services.go:1090: RemoveSnapServices - disabling snap.oracle-cloud-agent.oracle-cloud-agent-updater.service
Dec 20 08:34:08 master-20221214 snapd[801]: storehelpers.go:748: cannot refresh: snap has no updates available: "core18", "oracle-cloud-agent"
Dec 20 12:58:40 master-20221214 snapd[801]: storehelpers.go:748: cannot refresh: snap has no updates available: "core18", "core20", "lxd", "oracle-cloud-agent", "snapd"
Dec 20 12:58:40 master-20221214 snapd[801]: autorefresh.go:540: auto-refresh: all snaps are up-to-date
Dec 20 13:25:17 master-20221214 systemd[1]: Stopping Snap Daemon...
Dec 20 13:25:17 master-20221214 snapd[801]: main.go:155: Exiting on terminated signal.
Dec 20 13:25:17 master-20221214 snapd[801]: overlord.go:504: Released state lock file
Dec 20 13:25:17 master-20221214 systemd[1]: snapd.service: Succeeded.
Dec 20 13:25:17 master-20221214 systemd[1]: Stopped Snap Daemon.
```

確認
```
$ systemctl list-units --type=service | grep snapd
  snapd.apparmor.service                                     loaded active exited  Load AppArmor profiles managed internally by snapd
  snapd.seeded.service                                       loaded active exited  Wait until snapd is fully seeded
$ systemctl list-units --type=service --state=running | grep snapd
$
```

無効化
```
$ sudo systemctl disable snapd
Removed /etc/systemd/system/multi-user.target.wants/snapd.service.
```

## サービスのインストールと調査

ここではvsftpdとsambaをインストールする。
```
$ sudo apt-get install vsftpd samba
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  attr ibverbs-providers libavahi-client3 libavahi-common-data libavahi-common3 libboost-iostreams1.71.0 libboost-thread1.71.0 libcephfs2 libcups2 libibverbs1 libjansson4 libldb2 libnl-3-200 libnl-route-3-200 librados2 librdmacm1
  libtalloc2 libtevent0 libwbclient0 python3-crypto python3-dnspython python3-gpg python3-ldb python3-markdown python3-packaging python3-pygments python3-pyparsing python3-samba python3-talloc python3-tdb samba-common samba-common-bin
  samba-dsdb-modules samba-libs samba-vfs-modules ssl-cert tdb-tools
Suggested packages:
  cups-common python-markdown-doc python-pygments-doc ttf-bitstream-vera python-pyparsing-doc bind9 bind9utils ctdb ldb-tools ntp | chrony smbldap-tools winbind heimdal-clients openssl-blacklist
The following NEW packages will be installed:
  attr ibverbs-providers libavahi-client3 libavahi-common-data libavahi-common3 libboost-iostreams1.71.0 libboost-thread1.71.0 libcephfs2 libcups2 libibverbs1 libjansson4 libldb2 libnl-3-200 libnl-route-3-200 librados2 librdmacm1
  libtalloc2 libtevent0 libwbclient0 python3-crypto python3-dnspython python3-gpg python3-ldb python3-markdown python3-packaging python3-pygments python3-pyparsing python3-samba python3-talloc python3-tdb samba samba-common
  samba-common-bin samba-dsdb-modules samba-libs samba-vfs-modules ssl-cert tdb-tools vsftpd
0 upgraded, 39 newly installed, 0 to remove and 18 not upgraded.
Need to get 17.5 MB of archives.
After this operation, 102 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
・・・
samba-ad-dc.service is a disabled or a static unit, not starting it.
Processing triggers for ufw (0.36-6ubuntu1) ...
Processing triggers for systemd (245.4-4ubuntu3.18) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
```

```
$ systemctl status vsftpd.service
● vsftpd.service - vsftpd FTP server
     Loaded: loaded (/lib/systemd/system/vsftpd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-12-21 12:54:37 UTC; 1min 48s ago
   Main PID: 10396 (vsftpd)
      Tasks: 1 (limit: 9444)
     Memory: 520.0K
     CGroup: /system.slice/vsftpd.service
             mq10396 /usr/sbin/vsftpd /etc/vsftpd.conf

Dec 21 12:54:37 master-20221214 systemd[1]: Starting vsftpd FTP server...
Dec 21 12:54:37 master-20221214 systemd[1]: Started vsftpd FTP server.
$ systemctl status smbd.service
● smbd.service - Samba SMB Daemon
     Loaded: loaded (/lib/systemd/system/smbd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-12-21 12:54:43 UTC; 2min 21s ago
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
   Main PID: 11016 (smbd)
     Status: "smbd: ready to serve connections..."
      Tasks: 4 (limit: 9444)
     Memory: 9.6M
     CGroup: /system.slice/smbd.service
             tq11016 /usr/sbin/smbd --foreground --no-process-group
             tq11018 /usr/sbin/smbd --foreground --no-process-group
             tq11019 /usr/sbin/smbd --foreground --no-process-group
             mq11021 /usr/sbin/smbd --foreground --no-process-group

Dec 21 12:54:43 master-20221214 systemd[1]: Starting Samba SMB Daemon...
Dec 21 12:54:43 master-20221214 update-apparmor-samba-profile[11010]: grep: /etc/apparmor.d/samba/smbd-shares: No such file or directory
Dec 21 12:54:43 master-20221214 update-apparmor-samba-profile[11013]: diff: /etc/apparmor.d/samba/smbd-shares: No such file or directory
Dec 21 12:54:43 master-20221214 systemd[1]: Started Samba SMB Daemon.
```

プロセスを確認する。

```
$ ps aux | grep vsftpd
root       10396  0.0  0.0   6816  2968 ?        Ss   12:54   0:00 /usr/sbin/vsftpd /etc/vsftpd.conf
ubuntu     13498  0.0  0.0   8168   720 pts/0    S+   12:58   0:00 grep --color=auto vsftpd
$ ps aux | grep smbd
root       11016  0.0  0.3  86676 25860 ?        Ss   12:54   0:00 /usr/sbin/smbd --foreground --no-process-group
root       11018  0.0  0.1  84184 10408 ?        S    12:54   0:00 /usr/sbin/smbd --foreground --no-process-group
root       11019  0.0  0.0  84176  5912 ?        S    12:54   0:00 /usr/sbin/smbd --foreground --no-process-group
root       11021  0.0  0.1  86676 10504 ?        S    12:54   0:00 /usr/sbin/smbd --foreground --no-process-group
ubuntu     13810  0.0  0.0   8168   720 pts/0    S+   12:58   0:00 grep --color=auto smbd
```

vsftpdとsambaが使用しているポートとIPアドレスを確認する。
デフォルトでは全てのアドレスからのアクセスを許可している。

```
$ sudo netstat -plnt |grep -e smbd -e vsftpd
tcp        0      0 0.0.0.0:445             0.0.0.0:*               LISTEN      11016/smbd
tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN      11016/smbd
tcp6       0      0 :::445                  :::*                    LISTEN      11016/smbd
tcp6       0      0 :::139                  :::*                    LISTEN      11016/smbd
tcp6       0      0 :::21                   :::*                    LISTEN      10396/vsftpd
```

`/etc/samba/smb.conf`ファイルを編集して、サービスをrestartする。
以下の2行のコメントを外す。

- interfaces = 127.0.0.0/8 eth0
- bind interfaces only = yes

```
$ sudo vim /etc/samba/smb.conf
$ sudo systemctl restart smbd.service
$ sudo netstat -plnt |grep smbd
tcp        0      0 127.0.0.1:445           0.0.0.0:*               LISTEN      19450/smbd
tcp        0      0 127.0.0.1:139           0.0.0.0:*               LISTEN      19450/smbd
```

ローカルホストからのアクセスのみを許可するように変わってる。