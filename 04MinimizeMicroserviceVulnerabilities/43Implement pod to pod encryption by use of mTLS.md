# マニュアル

https://kubernetes.io/ja/docs/tasks/configure-pod-container/security-context/

# 学習ログ
テスト用Podを作成する。

```app.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: app
  name: app
spec:
  containers:
  - command:
    - sh
    - -c
    - ping google.com
    image: bash
    name: app
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
$ kubectl apply -f app.yaml
pod/app created
$ kubectl get pod
NAME   READY   STATUS    RESTARTS   AGE
app    1/1     Running   0          4s
```

```
$ kubectl logs app
PING google.com (172.253.63.138): 56 data bytes
64 bytes from 172.253.63.138: seq=0 ttl=109 time=1.524 ms
64 bytes from 172.253.63.138: seq=1 ttl=109 time=1.543 ms
64 bytes from 172.253.63.138: seq=2 ttl=109 time=1.582 ms
64 bytes from 172.253.63.138: seq=3 ttl=109 time=1.598 ms
64 bytes from 172.253.63.138: seq=4 ttl=109 time=1.525 ms
64 bytes from 172.253.63.138: seq=5 ttl=109 time=1.587 ms
64 bytes from 172.253.63.138: seq=6 ttl=109 time=1.540 ms
64 bytes from 172.253.63.138: seq=7 ttl=109 time=1.586 ms
64 bytes from 172.253.63.138: seq=8 ttl=109 time=1.572 ms
64 bytes from 172.253.63.138: seq=9 ttl=109 time=1.574 ms
64 bytes from 172.253.63.138: seq=10 ttl=109 time=1.584 ms
```

Proxyコンテナを追加して、Podを再デプロイする。

```app.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: app
  name: app
spec:
  containers:
  - command:
    - sh
    - -c
    - ping google.com
    image: bash
    name: app
    resources: {}
  - name: proxy
    image: ubuntu
    command:
    - sh
    - -c
    - "apt-get update && apt-get install iptables -y && iptables -L && sleep 1d"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
$ kubectl delete -f app.yaml
pod "app" deleted
$ kubectl apply -f app.yaml
pod/app created
$ kubectl get pod
NAME   READY   STATUS             RESTARTS      AGE
app    1/2     CrashLoopBackOff   3 (27s ago)   96s
```

コンテナが1つ起動に失敗してる。
ログを確認する。

```
$ kubectl logs app -c proxy
Get:1 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]
Get:2 http://archive.ubuntu.com/ubuntu jammy InRelease [270 kB]
Get:3 http://security.ubuntu.com/ubuntu jammy-security/restricted amd64 Packages [659 kB]
Get:4 http://security.ubuntu.com/ubuntu jammy-security/main amd64 Packages [720 kB]
Get:5 http://security.ubuntu.com/ubuntu jammy-security/multiverse amd64 Packages [4732 B]
Get:6 http://security.ubuntu.com/ubuntu jammy-security/universe amd64 Packages [786 kB]
Get:7 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [114 kB]
Get:8 http://archive.ubuntu.com/ubuntu jammy-backports InRelease [99.8 kB]
Get:9 http://archive.ubuntu.com/ubuntu jammy/main amd64 Packages [1792 kB]
Get:10 http://archive.ubuntu.com/ubuntu jammy/universe amd64 Packages [17.5 MB]
Get:11 http://archive.ubuntu.com/ubuntu jammy/restricted amd64 Packages [164 kB]
Get:12 http://archive.ubuntu.com/ubuntu jammy/multiverse amd64 Packages [266 kB]
Get:13 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages [1035 kB]
Get:14 http://archive.ubuntu.com/ubuntu jammy-updates/restricted amd64 Packages [708 kB]
Get:15 http://archive.ubuntu.com/ubuntu jammy-updates/multiverse amd64 Packages [8978 B]
Get:16 http://archive.ubuntu.com/ubuntu jammy-updates/universe amd64 Packages [990 kB]
Get:17 http://archive.ubuntu.com/ubuntu jammy-backports/universe amd64 Packages [7286 B]
Get:18 http://archive.ubuntu.com/ubuntu jammy-backports/main amd64 Packages [3520 B]
Fetched 25.2 MB in 3s (9122 kB/s)
Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed:
  libip4tc2 libip6tc2 libmnl0 libnetfilter-conntrack3 libnfnetlink0 libnftnl11
  libxtables12 netbase
Suggested packages:
  firewalld kmod nftables
The following NEW packages will be installed:
  iptables libip4tc2 libip6tc2 libmnl0 libnetfilter-conntrack3 libnfnetlink0
  libnftnl11 libxtables12 netbase
0 upgraded, 9 newly installed, 0 to remove and 0 not upgraded.
Need to get 678 kB of archives.
After this operation, 3708 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu jammy/main amd64 libip4tc2 amd64 1.8.7-1ubuntu5 [19.7 kB]
Get:2 http://archive.ubuntu.com/ubuntu jammy/main amd64 libmnl0 amd64 1.0.4-3build2 [13.2 kB]
Get:3 http://archive.ubuntu.com/ubuntu jammy/main amd64 libxtables12 amd64 1.8.7-1ubuntu5 [31.2 kB]
Get:4 http://archive.ubuntu.com/ubuntu jammy/main amd64 netbase all 6.3 [12.9 kB]
Get:5 http://archive.ubuntu.com/ubuntu jammy/main amd64 libip6tc2 amd64 1.8.7-1ubuntu5 [20.2 kB]
Get:6 http://archive.ubuntu.com/ubuntu jammy/main amd64 libnfnetlink0 amd64 1.0.1-3build3 [14.6 kB]
Get:7 http://archive.ubuntu.com/ubuntu jammy/main amd64 libnetfilter-conntrack3 amd64 1.0.9-1 [45.3 kB]
Get:8 http://archive.ubuntu.com/ubuntu jammy/main amd64 libnftnl11 amd64 1.2.1-1build1 [65.5 kB]
Get:9 http://archive.ubuntu.com/ubuntu jammy/main amd64 iptables amd64 1.8.7-1ubuntu5 [455 kB]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 678 kB in 1s (745 kB/s)
Selecting previously unselected package libip4tc2:amd64.
(Reading database ... 4395 files and directories currently installed.)
Preparing to unpack .../0-libip4tc2_1.8.7-1ubuntu5_amd64.deb ...
Unpacking libip4tc2:amd64 (1.8.7-1ubuntu5) ...
Selecting previously unselected package libmnl0:amd64.
Preparing to unpack .../1-libmnl0_1.0.4-3build2_amd64.deb ...
Unpacking libmnl0:amd64 (1.0.4-3build2) ...
Selecting previously unselected package libxtables12:amd64.
Preparing to unpack .../2-libxtables12_1.8.7-1ubuntu5_amd64.deb ...
Unpacking libxtables12:amd64 (1.8.7-1ubuntu5) ...
Selecting previously unselected package netbase.
Preparing to unpack .../3-netbase_6.3_all.deb ...
Unpacking netbase (6.3) ...
Selecting previously unselected package libip6tc2:amd64.
Preparing to unpack .../4-libip6tc2_1.8.7-1ubuntu5_amd64.deb ...
Unpacking libip6tc2:amd64 (1.8.7-1ubuntu5) ...
Selecting previously unselected package libnfnetlink0:amd64.
Preparing to unpack .../5-libnfnetlink0_1.0.1-3build3_amd64.deb ...
Unpacking libnfnetlink0:amd64 (1.0.1-3build3) ...
Selecting previously unselected package libnetfilter-conntrack3:amd64.
Preparing to unpack .../6-libnetfilter-conntrack3_1.0.9-1_amd64.deb ...
Unpacking libnetfilter-conntrack3:amd64 (1.0.9-1) ...
Selecting previously unselected package libnftnl11:amd64.
Preparing to unpack .../7-libnftnl11_1.2.1-1build1_amd64.deb ...
Unpacking libnftnl11:amd64 (1.2.1-1build1) ...
Selecting previously unselected package iptables.
Preparing to unpack .../8-iptables_1.8.7-1ubuntu5_amd64.deb ...
Unpacking iptables (1.8.7-1ubuntu5) ...
Setting up libip4tc2:amd64 (1.8.7-1ubuntu5) ...
Setting up libip6tc2:amd64 (1.8.7-1ubuntu5) ...
Setting up libmnl0:amd64 (1.0.4-3build2) ...
Setting up libxtables12:amd64 (1.8.7-1ubuntu5) ...
Setting up libnfnetlink0:amd64 (1.0.1-3build3) ...
Setting up netbase (6.3) ...
Setting up libnftnl11:amd64 (1.2.1-1build1) ...
Setting up libnetfilter-conntrack3:amd64 (1.0.9-1) ...
Setting up iptables (1.8.7-1ubuntu5) ...
update-alternatives: using /usr/sbin/iptables-legacy to provide /usr/sbin/iptables (iptables) in auto mode
update-alternatives: using /usr/sbin/ip6tables-legacy to provide /usr/sbin/ip6tables (ip6tables) in auto mode
update-alternatives: using /usr/sbin/iptables-nft to provide /usr/sbin/iptables (iptables) in auto mode
update-alternatives: using /usr/sbin/ip6tables-nft to provide /usr/sbin/ip6tables (ip6tables) in auto mode
update-alternatives: using /usr/sbin/arptables-nft to provide /usr/sbin/arptables (arptables) in auto mode
update-alternatives: using /usr/sbin/ebtables-nft to provide /usr/sbin/ebtables (ebtables) in auto mode
Processing triggers for libc-bin (2.35-0ubuntu3.1) ...
iptables v1.8.7 (nf_tables): Could not fetch rule set generation id: Permission denied (you must be root)
```

iptables が Permission denied

SecurityContextでNET_ADMIN権限を付与して再作成する。

```app.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: app
  name: app
spec:
  containers:
  - command:
    - sh
    - -c
    - ping google.com
    image: bash
    name: app
    resources: {}
  - name: proxy
    image: ubuntu
    command:
    - sh
    - -c
    - "apt-get update && apt-get install iptables -y && iptables -L && sleep 1d"
    securityContext:
      capabilities:
        add: ["NET_ADMIN"]
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
$ kubectl delete -f app.yaml
pod "app" deleted
$ kubectl apply -f app.yaml
pod/app created
$ kubectl get pod
NAME   READY   STATUS    RESTARTS   AGE
app    2/2     Running   0          26s
```

```
$ kubectl logs app -f proxy
Get:1 http://archive.ubuntu.com/ubuntu jammy InRelease [270 kB]
Get:2 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]
・・・
update-alternatives: using /usr/sbin/ebtables-nft to provide /usr/sbin/ebtables (ebtables) in auto mode
Processing triggers for libc-bin (2.35-0ubuntu3.1) ...
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```