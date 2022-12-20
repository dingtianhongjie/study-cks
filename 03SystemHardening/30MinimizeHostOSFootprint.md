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