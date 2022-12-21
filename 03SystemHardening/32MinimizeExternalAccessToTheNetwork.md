# マニュアル

# 学習ログ

ここでは21番ポートを使ってるサービスを確認して、停止する。

```
# netstat -plnt | grep 21
tcp6       0      0 :::21                   :::*                    LISTEN      10396/vsftpd
# lsof -i :21
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
vsftpd  10396 root    3u  IPv6  81839      0t0  TCP *:ftp (LISTEN)
```

```
# systemctl list-units --type service | grep vsftp
  vsftpd.service                                             loaded active running vsftpd FTP server
# systemctl status vsftpd.service
● vsftpd.service - vsftpd FTP server
     Loaded: loaded (/lib/systemd/system/vsftpd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-12-21 12:54:37 UTC; 33min ago
   Main PID: 10396 (vsftpd)
      Tasks: 1 (limit: 9444)
     Memory: 520.0K
     CGroup: /system.slice/vsftpd.service
             mq10396 /usr/sbin/vsftpd /etc/vsftpd.conf

Dec 21 12:54:37 master-20221214 systemd[1]: Starting vsftpd FTP server...
Dec 21 12:54:37 master-20221214 systemd[1]: Started vsftpd FTP server.
# systemctl stop vsftpd.service
# systemctl status vsftpd.service
● vsftpd.service - vsftpd FTP server
     Loaded: loaded (/lib/systemd/system/vsftpd.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Wed 2022-12-21 13:28:11 UTC; 1s ago
    Process: 10396 ExecStart=/usr/sbin/vsftpd /etc/vsftpd.conf (code=killed, signal=TERM)
   Main PID: 10396 (code=killed, signal=TERM)

Dec 21 12:54:37 master-20221214 systemd[1]: Starting vsftpd FTP server...
Dec 21 12:54:37 master-20221214 systemd[1]: Started vsftpd FTP server.
Dec 21 13:28:11 master-20221214 systemd[1]: Stopping vsftpd FTP server...
Dec 21 13:28:11 master-20221214 systemd[1]: vsftpd.service: Succeeded.
Dec 21 13:28:11 master-20221214 systemd[1]: Stopped vsftpd FTP server.
```

```
# netstat -plnt | grep 21
#
```