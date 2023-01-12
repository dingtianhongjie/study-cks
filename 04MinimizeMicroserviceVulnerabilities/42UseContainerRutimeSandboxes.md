# マニュアル
https://kubernetes.io/ja/docs/concepts/containers/runtime-class/

# 学習ログ
## コンテナからLinuxにsyscallできることを確認

Podを作成して、コンテナにログインし、カーネルバージョンを確認する。

```
$ kubectl run pod --image nginx
pod/pod created
$ kubectl exec -it pod -- bash
root@pod:/# uname -r
5.15.0-1021-oracle
root@pod:/# exit
exit
```

Master nodeでも同じように確認できる。
→コンテナでもLinux上でも同じsyscallができる。

```
$ uname -r
5.15.0-1021-oracle
```

syscallの詳細をトレースする。

```
$ strace uname -r
execve("/usr/bin/uname", ["uname", "-r"], 0x7ffdede2db08 /* 24 vars */) = 0
brk(NULL)                               = 0x561493b94000
arch_prctl(0x3001 /* ARCH_??? */, 0x7ffc799fe5b0) = -1 EINVAL (Invalid argument)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=25643, ...}) = 0
mmap(NULL, 25643, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f4053e56000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\300A\2\0\0\0\0\0"..., 832) = 832
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
pread64(3, "\4\0\0\0\20\0\0\0\5\0\0\0GNU\0\2\0\0\300\4\0\0\0\3\0\0\0\0\0\0\0", 32, 848) = 32
pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0\30x\346\264ur\f|Q\226\236i\253-'o"..., 68, 880) = 68
fstat(3, {st_mode=S_IFREG|0755, st_size=2029592, ...}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f4053e54000
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
pread64(3, "\4\0\0\0\20\0\0\0\5\0\0\0GNU\0\2\0\0\300\4\0\0\0\3\0\0\0\0\0\0\0", 32, 848) = 32
pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0\30x\346\264ur\f|Q\226\236i\253-'o"..., 68, 880) = 68
mmap(NULL, 2037344, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f4053c62000
mmap(0x7f4053c84000, 1540096, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x22000) = 0x7f4053c84000
mmap(0x7f4053dfc000, 319488, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x19a000) = 0x7f4053dfc000
mmap(0x7f4053e4a000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1e7000) = 0x7f4053e4a000
mmap(0x7f4053e50000, 13920, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f4053e50000
close(3)                                = 0
arch_prctl(ARCH_SET_FS, 0x7f4053e55580) = 0
mprotect(0x7f4053e4a000, 16384, PROT_READ) = 0
mprotect(0x56149278a000, 4096, PROT_READ) = 0
mprotect(0x7f4053e8a000, 4096, PROT_READ) = 0
munmap(0x7f4053e56000, 25643)           = 0
brk(NULL)                               = 0x561493b94000
brk(0x561493bb5000)                     = 0x561493bb5000
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=3035952, ...}) = 0
mmap(NULL, 3035952, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f405397c000
close(3)                                = 0
openat(AT_FDCWD, "/usr/share/locale/locale.alias", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=2996, ...}) = 0
read(3, "# Locale name alias data base.\n#"..., 4096) = 2996
read(3, "", 4096)                       = 0
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_IDENTIFICATION", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=252, ...}) = 0
mmap(NULL, 252, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f4053e89000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=27002, ...}) = 0
mmap(NULL, 27002, PROT_READ, MAP_SHARED, 3, 0) = 0x7f4053e56000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MEASUREMENT", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=23, ...}) = 0
mmap(NULL, 23, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f405397b000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_TELEPHONE", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=47, ...}) = 0
mmap(NULL, 47, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f405397a000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_ADDRESS", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=131, ...}) = 0
mmap(NULL, 131, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f4053979000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_NAME", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=62, ...}) = 0
mmap(NULL, 62, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f4053978000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_PAPER", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=34, ...}) = 0
mmap(NULL, 34, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f4053977000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MESSAGES", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MESSAGES/SYS_LC_MESSAGES", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=48, ...}) = 0
mmap(NULL, 48, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f4053976000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MONETARY", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=270, ...}) = 0
mmap(NULL, 270, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f4053975000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_COLLATE", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=1518110, ...}) = 0
mmap(NULL, 1518110, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f4053802000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_TIME", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=3360, ...}) = 0
mmap(NULL, 3360, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f4053801000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_NUMERIC", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=50, ...}) = 0
mmap(NULL, 50, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f4053800000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_CTYPE", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=201272, ...}) = 0
mmap(NULL, 201272, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f40537ce000
close(3)                                = 0
uname({sysname="Linux", nodename="master-20230109", ...}) = 0
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0), ...}) = 0
write(1, "5.15.0-1021-oracle\n", 195.15.0-1021-oracle
)    = 19
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```

## runtimeClassの作成

```runtime.yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc
```
```
$ kubectl apply -f runtime.yaml
runtimeclass.node.k8s.io/gvisor created
$ kubectl get runtimeclasses.node.k8s.io
NAME     HANDLER   AGE
gvisor   runsc     32s
```

作成したruntimeClassを指定してPodを作成する。

```pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: gvisor
  name: gvisor
spec:
  runtimeClassName: gvisor
  containers:
  - image: nginx
    name: gvisor
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
$ kubectl apply -f pod.yaml
pod/gvisor created
$ kubectl get pod
NAME     READY   STATUS              RESTARTS   AGE
gvisor   0/1     ContainerCreating   0          64s
```

詳細を確認する。

```
$ kubectl describe pod gvisor
Name:                gvisor
・・・
Events:
  Type     Reason                  Age                  From               Message
  ----     ------                  ----                 ----               -------
  Normal   Scheduled               2m21s                default-scheduler  Successfully assigned default/gvisor to worker-20230109
  Warning  FailedCreatePodSandBox  2s (x12 over 2m21s)  kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed to get sandbox runtime: no runtime for "runsc" is configured
```

まだgVisorをインストールしてないので、エラーになる。

## gVisorのインストール
WorkerノードでgVisorをインストールする。

```
worker$ bash <(curl -s https://raw.githubusercontent.com/killer-sh/cks-course-environment/master/course-content/microservice-vulnerabilities/container-runtimes/gvisor/install_gvisor.sh)
Hit:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Hit:2 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal InRelease
Hit:3 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:4 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:5 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
ca-certificates is already the newest version (20211016ubuntu0.20.04.1).
curl is already the newest version (7.68.0-1ubuntu2.15).
software-properties-common is already the newest version (0.99.9.8).
apt-transport-https is already the newest version (2.0.9).
gnupg-agent is already the newest version (2.2.19-3ubuntu2.2).
0 upgraded, 0 newly installed, 0 to remove and 26 not upgraded.
--2023-01-12 13:03:22--  https://storage.googleapis.com/gvisor/releases/release/20210806/x86_64/runsc
Resolving storage.googleapis.com (storage.googleapis.com)... 172.253.62.128, 142.251.16.128, 142.251.167.128, ...
Connecting to storage.googleapis.com (storage.googleapis.com)|172.253.62.128|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 35074107 (33M) [application/octet-stream]
Saving to: ‘runsc’

runsc                                                       100%[========================================================================================================================================>]  33.45M   161MB/s    in 0.2s

2023-01-12 13:03:22 (161 MB/s) - ‘runsc’ saved [35074107/35074107]

--2023-01-12 13:03:22--  https://storage.googleapis.com/gvisor/releases/release/20210806/x86_64/runsc.sha512
Reusing existing connection to storage.googleapis.com:443.
HTTP request sent, awaiting response... 200 OK
Length: 136 [application/octet-stream]
Saving to: ‘runsc.sha512’

runsc.sha512                                                100%[========================================================================================================================================>]     136  --.-KB/s    in 0s

2023-01-12 13:03:22 (62.3 MB/s) - ‘runsc.sha512’ saved [136/136]

--2023-01-12 13:03:22--  https://storage.googleapis.com/gvisor/releases/release/20210806/x86_64/containerd-shim-runsc-v1
Reusing existing connection to storage.googleapis.com:443.
HTTP request sent, awaiting response... 200 OK
Length: 24627868 (23M) [application/octet-stream]
Saving to: ‘containerd-shim-runsc-v1’

containerd-shim-runsc-v1                                    100%[========================================================================================================================================>]  23.49M   129MB/s    in 0.2s

2023-01-12 13:03:22 (129 MB/s) - ‘containerd-shim-runsc-v1’ saved [24627868/24627868]

--2023-01-12 13:03:22--  https://storage.googleapis.com/gvisor/releases/release/20210806/x86_64/containerd-shim-runsc-v1.sha512
Reusing existing connection to storage.googleapis.com:443.
HTTP request sent, awaiting response... 200 OK
Length: 155 [application/octet-stream]
Saving to: ‘containerd-shim-runsc-v1.sha512’

containerd-shim-runsc-v1.sha512                             100%[========================================================================================================================================>]     155  --.-KB/s    in 0s

2023-01-12 13:03:22 (59.5 MB/s) - ‘containerd-shim-runsc-v1.sha512’ saved [155/155]

FINISHED --2023-01-12 13:03:22--
Total wall clock time: 0.8s
Downloaded: 4 files, 57M in 0.4s (146 MB/s)
runsc: OK
containerd-shim-runsc-v1: OK
```

Podを確認すると、Runningになっている。

```
$ kubectl get pod
NAME     READY   STATUS    RESTARTS   AGE
gvisor   1/1     Running   0          9m46s
```

Podからカーネルバージョンを確認する。
→gVisorがエミュレートしたカーネルバージョンが表示される。

```
$ kubectl exec -it gvisor -- uname -r
4.4.0
```

```
$ kubectl get node -o wide
NAME              STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
master-20230109   Ready    control-plane   3d    v1.25.4   10.0.0.88     <none>        Ubuntu 20.04.5 LTS   5.15.0-1025-oracle   containerd://1.6.12
worker-20230109   Ready    <none>          3d    v1.25.4   10.0.0.140    <none>        Ubuntu 20.04.5 LTS   5.15.0-1025-oracle   containerd://1.6.12
```

gVisorが起動していることも確認できる。

```
$ kubectl exec -it gvisor -- dmesg
[    0.000000] Starting gVisor...
[    0.521694] Reticulating splines...
[    0.595901] Rewriting operating system in Javascript...
[    0.770972] Letting the watchdogs out...
[    1.243697] Conjuring /dev/null black hole...
[    1.710787] Creating process schedule...
[    2.137937] Reading process obituaries...
[    2.580221] Synthesizing system calls...
[    2.960449] Feeding the init monster...
[    3.455475] Waiting for children...
[    3.586123] Recruiting cron-ies...
[    3.843952] Ready!
```