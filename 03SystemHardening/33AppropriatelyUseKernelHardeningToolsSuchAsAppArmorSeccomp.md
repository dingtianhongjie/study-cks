# マニュアル
https://kubernetes.io/docs/tutorials/security/apparmor/
https://gitlab.com/apparmor/apparmor/-/wikis/Documentation

# 学習ログ
アプリケーションやLibraryが動作するUser SpaceとKernel Spaceの間に監視するハードニングツールを導入する。

## AppArmor
### 基本的な使い方

Worker nodeでcurlを使って外部と疎通できることを確認する。

```
worker$ curl goole.com -v
*   Trying 217.160.0.201:80...
* TCP_NODELAY set
* Connected to goole.com (217.160.0.201) port 80 (#0)
> GET / HTTP/1.1
> Host: goole.com
> User-Agent: curl/7.68.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 301 Moved Permanently
< Content-Type: text/html; charset=UTF-8
< Transfer-Encoding: chunked
< Connection: keep-alive
< Keep-Alive: timeout=15
< Date: Sun, 25 Dec 2022 12:29:17 GMT
< Server: Apache
< X-Pingback: http://www.goole.com/xmlrpc.php
< X-Redirect-By: WordPress
< Location: http://www.goole.com/
<
* Connection #0 to host goole.com left intact
```

AppArmorのデフォルトのステータスを確認する。

```
worker$ sudo aa-status
apparmor module is loaded.
35 profiles are loaded.
30 profiles are in enforce mode.
   /snap/snapd/17883/usr/lib/snapd/snap-confine
   /snap/snapd/17883/usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   /usr/bin/man
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/NetworkManager/nm-dhcp-helper
   /usr/lib/connman/scripts/dhclient-script
   /usr/lib/snapd/snap-confine
   /usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   /usr/sbin/tcpdump
   /{,usr/}sbin/dhclient
   cri-containerd.apparmor.d
   lsb_release
   man_filter
   man_groff
   nvidia_modprobe
   nvidia_modprobe//kmod
   snap-update-ns.lxd
   snap-update-ns.oracle-cloud-agent
   snap.lxd.activate
   snap.lxd.benchmark
   snap.lxd.buginfo
   snap.lxd.check-kernel
   snap.lxd.daemon
   snap.lxd.hook.configure
   snap.lxd.hook.install
   snap.lxd.hook.remove
   snap.lxd.lxc
   snap.lxd.lxc-to-lxd
   snap.lxd.lxd
   snap.lxd.migrate
5 profiles are in complain mode.
   snap.oracle-cloud-agent.hook.install
   snap.oracle-cloud-agent.hook.post-refresh
   snap.oracle-cloud-agent.hook.remove
   snap.oracle-cloud-agent.oracle-cloud-agent
   snap.oracle-cloud-agent.oracle-cloud-agent-updater
7 processes have profiles defined.
3 processes are in enforce mode.
   /coredns (2987) cri-containerd.apparmor.d
   /coredns (2995) cri-containerd.apparmor.d
   /usr/bin/kube-controllers (3249) cri-containerd.apparmor.d
4 processes are in complain mode.
   /snap/oracle-cloud-agent/48/agent (798) snap.oracle-cloud-agent.oracle-cloud-agent
   /snap/oracle-cloud-agent/48/plugins/gomon/gomon (1068) snap.oracle-cloud-agent.oracle-cloud-agent
   /snap/oracle-cloud-agent/48/plugins/unifiedmonitoring/unifiedmonitoring (1104) snap.oracle-cloud-agent.oracle-cloud-agent
   /snap/oracle-cloud-agent/48/updater/updater (796) snap.oracle-cloud-agent.oracle-cloud-agent-updater
0 processes are unconfined but have a profile defined.
```

35個のProfileがロードされている。

AppArmor Utilsをインストールする。

```
worker$ sudo apt-get install apparmor-utils
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  python3-apparmor python3-libapparmor
Suggested packages:
  vim-addon-manager
The following NEW packages will be installed:
  apparmor-utils python3-apparmor python3-libapparmor
0 upgraded, 3 newly installed, 0 to remove and 17 not upgraded.
Need to get 157 kB of archives.
After this operation, 966 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 python3-libapparmor amd64 2.13.3-7ubuntu5.1 [26.7 kB]
Get:2 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 python3-apparmor amd64 2.13.3-7ubuntu5.1 [78.6 kB]
Get:3 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 apparmor-utils amd64 2.13.3-7ubuntu5.1 [51.4 kB]
Fetched 157 kB in 0s (1566 kB/s)
Selecting previously unselected package python3-libapparmor.
(Reading database ... 144483 files and directories currently installed.)
Preparing to unpack .../python3-libapparmor_2.13.3-7ubuntu5.1_amd64.deb ...
Unpacking python3-libapparmor (2.13.3-7ubuntu5.1) ...
Selecting previously unselected package python3-apparmor.
Preparing to unpack .../python3-apparmor_2.13.3-7ubuntu5.1_amd64.deb ...
Unpacking python3-apparmor (2.13.3-7ubuntu5.1) ...
Selecting previously unselected package apparmor-utils.
Preparing to unpack .../apparmor-utils_2.13.3-7ubuntu5.1_amd64.deb ...
Unpacking apparmor-utils (2.13.3-7ubuntu5.1) ...
Setting up python3-libapparmor (2.13.3-7ubuntu5.1) ...
Setting up python3-apparmor (2.13.3-7ubuntu5.1) ...
Setting up apparmor-utils (2.13.3-7ubuntu5.1) ...
Processing triggers for man-db (2.9.1-1) ...
```

curlを監視するProfileを作成する。

```
worker$ sudo aa-genprof curl
Writing updated profile for /usr/bin/curl.
Setting /usr/bin/curl to complain mode.

Before you begin, you may wish to check if a
profile already exists for the application you
wish to confine. See the following wiki page for
more information:
https://gitlab.com/apparmor/apparmor/wikis/Profiles

Profiling: /usr/bin/curl

Please start the application to be profiled in
another window and exercise its functionality now.

Once completed, select the "Scan" option below in
order to scan the system logs for AppArmor events.

For each AppArmor event, you will be given the
opportunity to choose whether the access should be
allowed or denied.

[(S)can system log for AppArmor events] / (F)inish  ★Fを選択
Setting /usr/bin/curl to enforce mode.

Reloaded AppArmor profiles in enforce mode.

Please consider contributing your new profile!
See the following wiki page for more information:
https://gitlab.com/apparmor/apparmor/wikis/Profiles

Finished generating profile for /usr/bin/curl.
```

curlでの疎通を確認する。
→できなくなっている。

```
worker$ curl goole.com -v
* Could not resolve host: goole.com
* Closing connection 0
curl: (6) Could not resolve host: goole.com
```

statusを確認する。
→Profileが1つ増えていて、enforce modeに`/usr/bin/curl`が追加されている。

```
worker$ sudo aa-status
apparmor module is loaded.
36 profiles are loaded.
31 profiles are in enforce mode.
   /snap/snapd/17883/usr/lib/snapd/snap-confine
   /snap/snapd/17883/usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   /usr/bin/curl
   /usr/bin/man
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/NetworkManager/nm-dhcp-helper
   /usr/lib/connman/scripts/dhclient-script
   /usr/lib/snapd/snap-confine
   /usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   /usr/sbin/tcpdump
   /{,usr/}sbin/dhclient
   cri-containerd.apparmor.d
   lsb_release
   man_filter
   man_groff
   nvidia_modprobe
   nvidia_modprobe//kmod
   snap-update-ns.lxd
   snap-update-ns.oracle-cloud-agent
   snap.lxd.activate
   snap.lxd.benchmark
   snap.lxd.buginfo
   snap.lxd.check-kernel
   snap.lxd.daemon
   snap.lxd.hook.configure
   snap.lxd.hook.install
   snap.lxd.hook.remove
   snap.lxd.lxc
   snap.lxd.lxc-to-lxd
   snap.lxd.lxd
   snap.lxd.migrate
5 profiles are in complain mode.
   snap.oracle-cloud-agent.hook.install
   snap.oracle-cloud-agent.hook.post-refresh
   snap.oracle-cloud-agent.hook.remove
   snap.oracle-cloud-agent.oracle-cloud-agent
   snap.oracle-cloud-agent.oracle-cloud-agent-updater
7 processes have profiles defined.
3 processes are in enforce mode.
   /coredns (2987) cri-containerd.apparmor.d
   /coredns (2995) cri-containerd.apparmor.d
   /usr/bin/kube-controllers (3249) cri-containerd.apparmor.d
4 processes are in complain mode.
   /snap/oracle-cloud-agent/48/agent (798) snap.oracle-cloud-agent.oracle-cloud-agent
   /snap/oracle-cloud-agent/48/plugins/gomon/gomon (1068) snap.oracle-cloud-agent.oracle-cloud-agent
   /snap/oracle-cloud-agent/48/plugins/unifiedmonitoring/unifiedmonitoring (1104) snap.oracle-cloud-agent.oracle-cloud-agent
   /snap/oracle-cloud-agent/48/updater/updater (796) snap.oracle-cloud-agent.oracle-cloud-agent-updater
0 processes are unconfined but have a profile defined.
```

Profileの中身を確認する。

```
worker# cd /etc/apparmor.d/
worker# ls
abstractions  disable  force-complain  local  lsb_release  nvidia_modprobe  sbin.dhclient  tunables  usr.bin.curl  usr.bin.man  usr.lib.snapd.snap-confine.real  usr.sbin.rsyslogd  usr.sbin.tcpdump
worker# cat usr.bin.curl
# Last Modified: Sun Dec 25 12:38:12 2022
#include <tunables/global>

/usr/bin/curl {
  #include <abstractions/base>

  /usr/bin/curl mr,

}
```

Profileを更新する。

```
worker# aa-logprof
Reading log entries from /var/log/syslog.
Updating AppArmor profiles in /etc/apparmor.d.
Enforce-mode changes:

Profile:  /usr/bin/curl
Path:     /etc/ssl/openssl.cnf
New Mode: r
Severity: 2

 [1 - #include <abstractions/openssl>]
  2 - #include <abstractions/ssl_keys>
  3 - /etc/ssl/openssl.cnf r,
(A)llow / [(D)eny] / (I)gnore / (G)lob / Glob with (E)xtension / (N)ew / Audi(t) / Abo(r)t / (F)inish   ★Aを選択して、curlを許可する。
Adding #include <abstractions/openssl> to profile.

Profile:  /usr/bin/curl
Path:     /etc/host.conf
New Mode: r
Severity: unknown

= Changed Local Profiles =

The following local profiles were changed. Would you like to save them?

 [1 - /usr/bin/curl]
(S)ave Changes / Save Selec(t)ed Profile / [(V)iew Changes] / View Changes b/w (C)lean profiles / Abo(r)t
Writing updated profile for /usr/bin/curl.
```

更新されていることを確認する。

```
# cat usr.bin.curl
# Last Modified: Sun Dec 25 13:00:02 2022
#include <tunables/global>

/usr/bin/curl {
  #include <abstractions/base>
  #include <abstractions/nameservice>
  #include <abstractions/openssl>

  /usr/bin/curl mr,

}
```

curlが通るようになっている。

```
# curl google.com -v
*   Trying 172.253.62.100:80...
* TCP_NODELAY set
* Connected to google.com (172.253.62.100) port 80 (#0)
> GET / HTTP/1.1
> Host: google.com
> User-Agent: curl/7.68.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 301 Moved Permanently
< Location: http://www.google.com/
< Content-Type: text/html; charset=UTF-8
< Cross-Origin-Opener-Policy-Report-Only: same-origin-allow-popups; report-to="gws"
< Report-To: {"group":"gws","max_age":2592000,"endpoints":[{"url":"https://csp.withgoogle.com/csp/report-to/gws/other"}]}
< Date: Sun, 25 Dec 2022 13:03:51 GMT
< Expires: Tue, 24 Jan 2023 13:03:51 GMT
< Cache-Control: public, max-age=2592000
< Server: gws
< Content-Length: 219
< X-XSS-Protection: 0
< X-Frame-Options: SAMEORIGIN
<
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
* Connection #0 to host google.com left intact
```
### Dockerでの利用

nginx用のProfileを作成する。
以下の内容で`/etc/apparmor.d/docker-nginx`ファイルを作成する。

```/etc/apparmor.d/docker-nginx
#include <tunables/global>


profile docker-nginx flags=(attach_disconnected,mediate_deleted) {
  #include <abstractions/base>

  network inet tcp,
  network inet udp,
  network inet icmp,

  deny network raw,

  deny network packet,

  file,
  umount,

  deny /bin/** wl,
  deny /boot/** wl,
  deny /dev/** wl,
  deny /etc/** wl,
  deny /home/** wl,
  deny /lib/** wl,
  deny /lib64/** wl,
  deny /media/** wl,
  deny /mnt/** wl,
  deny /opt/** wl,
  deny /proc/** wl,
  deny /root/** wl,
  deny /sbin/** wl,
  deny /srv/** wl,
  deny /tmp/** wl,
  deny /sys/** wl,
  deny /usr/** wl,

  audit /** w,

  /var/run/nginx.pid w,

  /usr/sbin/nginx ix,

  deny /bin/dash mrwklx,
  deny /bin/sh mrwklx,
  deny /usr/bin/top mrwklx,


  capability chown,
  capability dac_override,
  capability setuid,
  capability setgid,
  capability net_bind_service,

  deny @{PROC}/* w,   # deny write for all files directly in /proc (not in a subdir)
  # deny write to files not in /proc/<number>/** or /proc/sys/**
  deny @{PROC}/{[^1-9],[^1-9][^0-9],[^1-9s][^0-9y][^0-9s],[^1-9][^0-9][^0-9][^0-9]*}/** w,
  deny @{PROC}/sys/[^k]** w,  # deny /proc/sys except /proc/sys/k* (effectively /proc/sys/kernel)
  deny @{PROC}/sys/kernel/{?,??,[^s][^h][^m]**} w,  # deny everything except shm* in /proc/sys/kernel/
  deny @{PROC}/sysrq-trigger rwklx,
  deny @{PROC}/mem rwklx,
  deny @{PROC}/kmem rwklx,
  deny @{PROC}/kcore rwklx,

  deny mount,

  deny /sys/[^f]*/** wklx,
  deny /sys/f[^s]*/** wklx,
  deny /sys/fs/[^c]*/** wklx,
  deny /sys/fs/c[^g]*/** wklx,
  deny /sys/fs/cg[^r]*/** wklx,
  deny /sys/firmware/** rwklx,
  deny /sys/kernel/security/** rwklx,
}
```
Profileを読み込む

```
# apparmor_parser /etc/apparmor.d/docker-nginx
```

確認

```
# aa-status
apparmor module is loaded.
37 profiles are loaded.
32 profiles are in enforce mode.
   /snap/snapd/17883/usr/lib/snapd/snap-confine
   /snap/snapd/17883/usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   /usr/bin/curl
   /usr/bin/man
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/NetworkManager/nm-dhcp-helper
   /usr/lib/connman/scripts/dhclient-script
   /usr/lib/snapd/snap-confine
   /usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   /usr/sbin/tcpdump
   /{,usr/}sbin/dhclient
   cri-containerd.apparmor.d
   docker-nginx
   lsb_release
・・・
```

※Doker入れてないから動作確認できず。

## Seccomp