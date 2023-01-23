# マニュアル
https://falco.org/ja/docs/getting-started/installation/

# 学習ログ
## Strace

straceコマンドの引数で指定したコマンドのSyscallを確認できる。
```
$ ls /
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  lost+found  media  mnt  opt  proc  root  run  sbin  snap  srv  sys  tmp  usr  var

$ strace ls /
execve("/usr/bin/ls", ["ls", "/"], 0x7fff10847328 /* 24 vars */) = 0
brk(NULL)                               = 0x5633661b6000
arch_prctl(0x3001 /* ARCH_??? */, 0x7ffc082da490) = -1 EINVAL (Invalid argument)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=25643, ...}) = 0
mmap(NULL, 25643, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba42cf000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libselinux.so.1", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0@p\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0644, st_size=163200, ...}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fbba42cd000
mmap(NULL, 174600, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fbba42a2000
mprotect(0x7fbba42a8000, 135168, PROT_NONE) = 0
mmap(0x7fbba42a8000, 102400, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x6000) = 0x7fbba42a8000
mmap(0x7fbba42c1000, 28672, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1f000) = 0x7fbba42c1000
mmap(0x7fbba42c9000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x26000) = 0x7fbba42c9000
mmap(0x7fbba42cb000, 6664, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fbba42cb000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\300A\2\0\0\0\0\0"..., 832) = 832
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
pread64(3, "\4\0\0\0\20\0\0\0\5\0\0\0GNU\0\2\0\0\300\4\0\0\0\3\0\0\0\0\0\0\0", 32, 848) = 32
pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0\30x\346\264ur\f|Q\226\236i\253-'o"..., 68, 880) = 68
fstat(3, {st_mode=S_IFREG|0755, st_size=2029592, ...}) = 0
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
pread64(3, "\4\0\0\0\20\0\0\0\5\0\0\0GNU\0\2\0\0\300\4\0\0\0\3\0\0\0\0\0\0\0", 32, 848) = 32
pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0\30x\346\264ur\f|Q\226\236i\253-'o"..., 68, 880) = 68
mmap(NULL, 2037344, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fbba40b0000
mmap(0x7fbba40d2000, 1540096, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x22000) = 0x7fbba40d2000
mmap(0x7fbba424a000, 319488, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x19a000) = 0x7fbba424a000
mmap(0x7fbba4298000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1e7000) = 0x7fbba4298000
mmap(0x7fbba429e000, 13920, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fbba429e000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpcre2-8.so.0", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\340\"\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0644, st_size=588488, ...}) = 0
mmap(NULL, 590632, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fbba401f000
mmap(0x7fbba4021000, 413696, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x2000) = 0x7fbba4021000
mmap(0x7fbba4086000, 163840, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x67000) = 0x7fbba4086000
mmap(0x7fbba40ae000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x8e000) = 0x7fbba40ae000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libdl.so.2", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0 \22\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0644, st_size=18848, ...}) = 0
mmap(NULL, 20752, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fbba4019000
mmap(0x7fbba401a000, 8192, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1000) = 0x7fbba401a000
mmap(0x7fbba401c000, 4096, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x3000) = 0x7fbba401c000
mmap(0x7fbba401d000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x3000) = 0x7fbba401d000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\220q\0\0\0\0\0\0"..., 832) = 832
pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0{E6\364\34\332\245\210\204\10\350-\0106\343="..., 68, 824) = 68
fstat(3, {st_mode=S_IFREG|0755, st_size=157224, ...}) = 0
pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0{E6\364\34\332\245\210\204\10\350-\0106\343="..., 68, 824) = 68
mmap(NULL, 140408, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fbba3ff6000
mmap(0x7fbba3ffc000, 69632, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x6000) = 0x7fbba3ffc000
mmap(0x7fbba400d000, 24576, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x17000) = 0x7fbba400d000
mmap(0x7fbba4013000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1c000) = 0x7fbba4013000
mmap(0x7fbba4015000, 13432, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fbba4015000
close(3)                                = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fbba3ff4000
arch_prctl(ARCH_SET_FS, 0x7fbba3ff5400) = 0
mprotect(0x7fbba4298000, 16384, PROT_READ) = 0
mprotect(0x7fbba4013000, 4096, PROT_READ) = 0
mprotect(0x7fbba401d000, 4096, PROT_READ) = 0
mprotect(0x7fbba40ae000, 4096, PROT_READ) = 0
mprotect(0x7fbba42c9000, 4096, PROT_READ) = 0
mprotect(0x563365b68000, 4096, PROT_READ) = 0
mprotect(0x7fbba4303000, 4096, PROT_READ) = 0
munmap(0x7fbba42cf000, 25643)           = 0
set_tid_address(0x7fbba3ff56d0)         = 154428
set_robust_list(0x7fbba3ff56e0, 24)     = 0
rt_sigaction(SIGRTMIN, {sa_handler=0x7fbba3ffcbf0, sa_mask=[], sa_flags=SA_RESTORER|SA_SIGINFO, sa_restorer=0x7fbba400a420}, NULL, 8) = 0
rt_sigaction(SIGRT_1, {sa_handler=0x7fbba3ffcc90, sa_mask=[], sa_flags=SA_RESTORER|SA_RESTART|SA_SIGINFO, sa_restorer=0x7fbba400a420}, NULL, 8) = 0
rt_sigprocmask(SIG_UNBLOCK, [RTMIN RT_1], NULL, 8) = 0
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
statfs("/sys/fs/selinux", 0x7ffc082da3e0) = -1 ENOENT (No such file or directory)
statfs("/selinux", 0x7ffc082da3e0)      = -1 ENOENT (No such file or directory)
brk(NULL)                               = 0x5633661b6000
brk(0x5633661d7000)                     = 0x5633661d7000
openat(AT_FDCWD, "/proc/filesystems", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0444, st_size=0, ...}) = 0
read(3, "nodev\tsysfs\nnodev\ttmpfs\nnodev\tbd"..., 1024) = 497
read(3, "", 1024)                       = 0
close(3)                                = 0
access("/etc/selinux/config", F_OK)     = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=3035952, ...}) = 0
mmap(NULL, 3035952, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba3d0e000
close(3)                                = 0
openat(AT_FDCWD, "/usr/share/locale/locale.alias", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=2996, ...}) = 0
read(3, "# Locale name alias data base.\n#"..., 4096) = 2996
read(3, "", 4096)                       = 0
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_IDENTIFICATION", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=252, ...}) = 0
mmap(NULL, 252, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba4302000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=27002, ...}) = 0
mmap(NULL, 27002, PROT_READ, MAP_SHARED, 3, 0) = 0x7fbba42cf000
close(3)                                = 0
futex(0x7fbba429d954, FUTEX_WAKE_PRIVATE, 2147483647) = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MEASUREMENT", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=23, ...}) = 0
mmap(NULL, 23, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba3d0d000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_TELEPHONE", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=47, ...}) = 0
mmap(NULL, 47, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba3d0c000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_ADDRESS", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=131, ...}) = 0
mmap(NULL, 131, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba3d0b000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_NAME", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=62, ...}) = 0
mmap(NULL, 62, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba3d0a000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_PAPER", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=34, ...}) = 0
mmap(NULL, 34, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba3d09000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MESSAGES", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MESSAGES/SYS_LC_MESSAGES", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=48, ...}) = 0
mmap(NULL, 48, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba3d08000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MONETARY", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=270, ...}) = 0
mmap(NULL, 270, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba3d07000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_COLLATE", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=1518110, ...}) = 0
mmap(NULL, 1518110, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba3b94000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_TIME", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=3360, ...}) = 0
mmap(NULL, 3360, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba3b93000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_NUMERIC", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=50, ...}) = 0
mmap(NULL, 50, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba3b92000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_CTYPE", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=201272, ...}) = 0
mmap(NULL, 201272, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbba3b60000
close(3)                                = 0
ioctl(1, TCGETS, {B38400 opost isig icanon echo ...}) = 0
ioctl(1, TIOCGWINSZ, {ws_row=55, ws_col=237, ws_xpixel=1933, ws_ypixel=1053}) = 0
stat("/", {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
openat(AT_FDCWD, "/", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
fstat(3, {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
getdents64(3, /* 25 entries */, 32768)  = 640
getdents64(3, /* 0 entries */, 32768)   = 0
close(3)                                = 0
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0), ...}) = 0
write(1, "bin  boot  dev\tetc  home  lib\tli"..., 133bin  boot  dev     etc  home  lib  lib32  lib64  libx32  lost+found  media  mnt  opt  proc  root  run  sbin  snap  srv  sys  tmp  usr  var
) = 133
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```

`-cw`オプションでSyscallの一覧を表示できる。

```
$ strace -cw ls /
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  lost+found  media  mnt  opt  proc  root  run  sbin  snap  srv  sys  tmp  usr  var
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 23.20    0.000220           5        40           mmap
 14.26    0.000135           5        24           openat
 13.00    0.000123         123         1           execve
 11.00    0.000104           4        25           fstat
 10.91    0.000103           3        26           close
  5.95    0.000056           6         9           read
  5.04    0.000048           5         8           mprotect
  3.09    0.000029           3         8           pread64
  1.97    0.000019           9         2           getdents64
  1.83    0.000017          17         1           write
  1.49    0.000014           7         2         2 access
  1.47    0.000014           6         2         2 statfs
  1.23    0.000012           3         3           brk
  0.88    0.000008           4         2           ioctl
  0.82    0.000008           7         1           munmap
  0.77    0.000007           3         2         1 arch_prctl
  0.74    0.000007           3         2           rt_sigaction
  0.46    0.000004           4         1           stat
  0.40    0.000004           3         1           prlimit64
  0.39    0.000004           3         1           set_tid_address
  0.38    0.000004           3         1           futex
  0.37    0.000003           3         1           rt_sigprocmask
  0.36    0.000003           3         1           set_robust_list
------ ----------- ----------- --------- --------- ----------------
100.00    0.000948                   164         5 total
```

etcdのプロセスIDを確認する。

```
$ sudo crictl ps | grep etcd
f04a6d51203ed       4694d02f8e611       6 hours ago         Running             etcd                      0                   258ea8b4685f1       etcd-master-20230122

$ ps aux | grep etcd
root        2769  2.1  2.2 1114800 370904 ?      Ssl  06:32   8:18 kube-apiserver --advertise-address=10.0.0.195 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root        2792  1.1  0.4 11215236 67932 ?      Ssl  06:32   4:18 etcd --advertise-client-urls=https://10.0.0.195:2379 --cert-file=/etc/kubernetes/pki/etcd/server.crt --client-cert-auth=true --data-dir=/var/lib/etcd --experimental-initial-corrupt-check=true --experimental-watch-progress-notify-interval=5s --initial-advertise-peer-urls=https://10.0.0.195:2380 --initial-cluster=master-20230122=https://10.0.0.195:2380 --key-file=/etc/kubernetes/pki/etcd/server.key --listen-client-urls=https://127.0.0.1:2379,https://10.0.0.195:2379 --listen-metrics-urls=http://127.0.0.1:2381 --listen-peer-urls=https://10.0.0.195:2380 --name=master-20230122 --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt --peer-client-cert-auth=true --peer-key-file=/etc/kubernetes/pki/etcd/peer.key --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt --snapshot-count=10000 --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
ubuntu    157958  0.0  0.0   8168   720 pts/0    R+   13:01   0:00 grep --color=auto etcd
```

etcdのプロセスIDを指定して、Syscallを確認する。

```
$ sudo strace -p 2792
strace: Process 2792 attached
futex(0x1ab3f30, FUTEX_WAIT_PRIVATE, 0, NULL) = 0
futex(0x1ab3f30, FUTEX_WAIT_PRIVATE, 0, NULL) = 0
futex(0x1ab3f30, FUTEX_WAIT_PRIVATE, 0, NULL) = 0
futex(0x1ab3f30, FUTEX_WAIT_PRIVATE, 0, NULL) = 0
futex(0x1ab3f30, FUTEX_WAIT_PRIVATE, 0, NULL) = 0
epoll_pwait(4, [], 128, 0, NULL, 23641) = 0
write(75, "\27\3\3\0%\0\0\0\0\0\2$K\341\311w|\375\34\334\265\207\211\271\362\270\22\204\241\331\353J"..., 42) = 42
futex(0xc00005b150, FUTEX_WAKE_PRIVATE, 1) = 1
・・・
```

Syscallの統計を確認する。（少し待ってCtrl-Cする）

```
$ sudo strace -p 2792 -cw -f
strace: Process 2792 attached with 10 threads
^Cstrace: Process 2792 detached
strace: Process 2870 detached
strace: Process 2871 detached
strace: Process 2872 detached
strace: Process 2882 detached
strace: Process 2883 detached
strace: Process 2891 detached
strace: Process 2892 detached
strace: Process 2893 detached
strace: Process 3264 detached
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 75.49   83.602205       13499      6193      1131 futex
 20.17   22.333491        3165      7056         1 epoll_pwait
  4.19    4.638092         817      5675           nanosleep
  0.06    0.070651          54      1308           write
  0.05    0.052359         707        74           fdatasync
  0.03    0.027836          24      1133       561 read
  0.02    0.017605       17605         1           restart_syscall
  0.00    0.001880          18        99           pwrite64
  0.00    0.001632          31        51           tgkill
  0.00    0.001185          34        34           sched_yield
  0.00    0.001164          22        51           getpid
  0.00    0.000895          17        51         2 rt_sigreturn
  0.00    0.000608          17        34           lseek
  0.00    0.000288          13        21           setsockopt
  0.00    0.000221          27         8           close
  0.00    0.000167          10        16        10 epoll_ctl
  0.00    0.000165          32         5           openat
  0.00    0.000113          18         6         3 accept4
  0.00    0.000095          15         6           getdents64
  0.00    0.000073          24         3           getsockname
  0.00    0.000043          10         4           getrandom
  0.00    0.000036          17         2           fstat
------ ----------- ----------- --------- --------- ----------------
100.00  110.750803                 21831      1708 total
```

## Falco

### Install

Worker nodeにインストールする。
```
# curl -s https://falco.org/repo/falcosecurity-packages.asc | apt-key add -
OK
# echo "deb https://download.falco.org/packages/deb stable main" | tee -a /etc/apt/sources.list.d/falcosecurity.list
deb https://download.falco.org/packages/deb stable main
# apt-get update -y
Get:1 https://download.falco.org/packages/deb stable InRelease [4168 B]
Hit:2 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal InRelease
Hit:3 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:4 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-backports InRelease
Hit:6 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:5 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Get:7 https://download.falco.org/packages/deb stable/main amd64 Packages [4578 B]
Fetched 8746 B in 0s (20.7 kB/s)
Reading package lists... Done
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target CNF (main/cnf/Commands-amd64) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target CNF (main/cnf/Commands-all) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target CNF (main/cnf/Commands-amd64) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target CNF (main/cnf/Commands-all) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
# apt-get -y install linux-headers-$(uname -r)
Reading package lists... Done
Building dependency tree
Reading state information... Done
linux-headers-5.15.0-1027-oracle is already the newest version (5.15.0-1027.33~20.04.1).
0 upgraded, 0 newly installed, 0 to remove and 33 not upgraded.
# apt-get install -y falco
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  binutils binutils-common binutils-x86-64-linux-gnu build-essential cpp cpp-9 dctrl-tools dkms dpkg-dev fakeroot g++ g++-9 gcc gcc-9 gcc-9-base libalgorithm-diff-perl libalgorithm-diff-xs-perl libalgorithm-merge-perl libasan5
  libatomic1 libbinutils libc-dev-bin libc6-dev libcc1-0 libcrypt-dev libctf-nobfd0 libctf0 libdpkg-perl libfakeroot libfile-fcntllock-perl libgcc-9-dev libgomp1 libisl22 libitm1 liblsan0 libmpc3 libquadmath0 libstdc++-9-dev libtsan0
  libubsan1 linux-libc-dev make manpages-dev
Suggested packages:
  binutils-doc cpp-doc gcc-9-locales debtags menu debian-keyring g++-multilib g++-9-multilib gcc-9-doc gcc-multilib autoconf automake libtool flex bison gdb gcc-doc gcc-9-multilib glibc-doc bzr libstdc++-9-doc make-doc
The following NEW packages will be installed:
  binutils binutils-common binutils-x86-64-linux-gnu build-essential cpp cpp-9 dctrl-tools dkms dpkg-dev fakeroot falco g++ g++-9 gcc gcc-9 gcc-9-base libalgorithm-diff-perl libalgorithm-diff-xs-perl libalgorithm-merge-perl libasan5
  libatomic1 libbinutils libc-dev-bin libc6-dev libcc1-0 libcrypt-dev libctf-nobfd0 libctf0 libdpkg-perl libfakeroot libfile-fcntllock-perl libgcc-9-dev libgomp1 libisl22 libitm1 liblsan0 libmpc3 libquadmath0 libstdc++-9-dev libtsan0
  libubsan1 linux-libc-dev make manpages-dev
0 upgraded, 44 newly installed, 0 to remove and 33 not upgraded.
Need to get 63.6 MB of archives.
After this operation, 243 MB of additional disk space will be used.
Get:1 https://download.falco.org/packages/deb stable/main amd64 falco amd64 0.33.1 [18.1 MB]
Get:2 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 gcc-9-base amd64 9.4.0-1ubuntu1~20.04.1 [19.4 kB]
Get:3 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 libisl22 amd64 0.22.1-1 [592 kB]
Get:4 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 libmpc3 amd64 1.1.0-1 [40.8 kB]
Get:5 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 cpp-9 amd64 9.4.0-1ubuntu1~20.04.1 [7500 kB]
Get:6 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 cpp amd64 4:9.3.0-1ubuntu2 [27.6 kB]
Get:7 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libcc1-0 amd64 10.3.0-1ubuntu1~20.04 [48.8 kB]
Get:8 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 binutils-common amd64 2.34-6ubuntu1.4 [207 kB]
Get:9 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libbinutils amd64 2.34-6ubuntu1.4 [474 kB]
Get:10 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libctf-nobfd0 amd64 2.34-6ubuntu1.4 [47.2 kB]
Get:11 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libctf0 amd64 2.34-6ubuntu1.4 [46.6 kB]
Get:12 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 binutils-x86-64-linux-gnu amd64 2.34-6ubuntu1.4 [1613 kB]
Get:13 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 binutils amd64 2.34-6ubuntu1.4 [3380 B]
Get:14 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libgomp1 amd64 10.3.0-1ubuntu1~20.04 [102 kB]
Get:15 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libitm1 amd64 10.3.0-1ubuntu1~20.04 [26.2 kB]
Get:16 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libatomic1 amd64 10.3.0-1ubuntu1~20.04 [9284 B]
Get:17 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libasan5 amd64 9.4.0-1ubuntu1~20.04.1 [2751 kB]
Get:18 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 liblsan0 amd64 10.3.0-1ubuntu1~20.04 [835 kB]
Get:19 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libtsan0 amd64 10.3.0-1ubuntu1~20.04 [2009 kB]
Get:20 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libubsan1 amd64 10.3.0-1ubuntu1~20.04 [784 kB]
Get:21 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libquadmath0 amd64 10.3.0-1ubuntu1~20.04 [146 kB]
Get:22 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libgcc-9-dev amd64 9.4.0-1ubuntu1~20.04.1 [2359 kB]
Get:23 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 gcc-9 amd64 9.4.0-1ubuntu1~20.04.1 [8274 kB]
Get:24 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 gcc amd64 4:9.3.0-1ubuntu2 [5208 B]
Get:25 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libdpkg-perl all 1.19.7ubuntu3.2 [231 kB]
Get:26 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 make amd64 4.2.1-1.2 [162 kB]
Get:27 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 dpkg-dev all 1.19.7ubuntu3.2 [679 kB]
Get:28 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libc-dev-bin amd64 2.31-0ubuntu9.9 [71.8 kB]
Get:29 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 linux-libc-dev amd64 5.4.0-137.154 [1120 kB]
Get:30 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 libcrypt-dev amd64 1:4.4.10-10ubuntu4 [104 kB]
Get:31 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libc6-dev amd64 2.31-0ubuntu9.9 [2519 kB]
Get:32 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 libstdc++-9-dev amd64 9.4.0-1ubuntu1~20.04.1 [1722 kB]
Get:33 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 g++-9 amd64 9.4.0-1ubuntu1~20.04.1 [8420 kB]
Get:34 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 g++ amd64 4:9.3.0-1ubuntu2 [1604 B]
Get:35 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 build-essential amd64 12.8ubuntu1.1 [4664 B]
Get:36 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 dctrl-tools amd64 2.24-3 [61.5 kB]
Get:37 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates/main amd64 dkms all 2.8.1-5ubuntu2 [66.8 kB]
Get:38 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 libfakeroot amd64 1.24-1 [25.7 kB]
Get:39 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 fakeroot amd64 1.24-1 [62.6 kB]
Get:40 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 libalgorithm-diff-perl all 1.19.03-2 [46.6 kB]
Get:41 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 libalgorithm-diff-xs-perl amd64 0.04-6 [11.3 kB]
Get:42 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 libalgorithm-merge-perl all 0.08-3 [12.0 kB]
Get:43 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 libfile-fcntllock-perl amd64 0.22-3build4 [33.1 kB]
Get:44 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal/main amd64 manpages-dev all 5.05-1 [2266 kB]
Fetched 63.6 MB in 1s (96.6 MB/s)
Extracting templates from packages: 100%
Selecting previously unselected package gcc-9-base:amd64.
(Reading database ... 144483 files and directories currently installed.)
Preparing to unpack .../00-gcc-9-base_9.4.0-1ubuntu1~20.04.1_amd64.deb ...
Unpacking gcc-9-base:amd64 (9.4.0-1ubuntu1~20.04.1) ...
Selecting previously unselected package libisl22:amd64.
Preparing to unpack .../01-libisl22_0.22.1-1_amd64.deb ...
Unpacking libisl22:amd64 (0.22.1-1) ...
Selecting previously unselected package libmpc3:amd64.
Preparing to unpack .../02-libmpc3_1.1.0-1_amd64.deb ...
Unpacking libmpc3:amd64 (1.1.0-1) ...
Selecting previously unselected package cpp-9.
Preparing to unpack .../03-cpp-9_9.4.0-1ubuntu1~20.04.1_amd64.deb ...
Unpacking cpp-9 (9.4.0-1ubuntu1~20.04.1) ...
Selecting previously unselected package cpp.
Preparing to unpack .../04-cpp_4%3a9.3.0-1ubuntu2_amd64.deb ...
Unpacking cpp (4:9.3.0-1ubuntu2) ...
Selecting previously unselected package libcc1-0:amd64.
Preparing to unpack .../05-libcc1-0_10.3.0-1ubuntu1~20.04_amd64.deb ...
Unpacking libcc1-0:amd64 (10.3.0-1ubuntu1~20.04) ...
Selecting previously unselected package binutils-common:amd64.
Preparing to unpack .../06-binutils-common_2.34-6ubuntu1.4_amd64.deb ...
Unpacking binutils-common:amd64 (2.34-6ubuntu1.4) ...
Selecting previously unselected package libbinutils:amd64.
Preparing to unpack .../07-libbinutils_2.34-6ubuntu1.4_amd64.deb ...
Unpacking libbinutils:amd64 (2.34-6ubuntu1.4) ...
Selecting previously unselected package libctf-nobfd0:amd64.
Preparing to unpack .../08-libctf-nobfd0_2.34-6ubuntu1.4_amd64.deb ...
Unpacking libctf-nobfd0:amd64 (2.34-6ubuntu1.4) ...
Selecting previously unselected package libctf0:amd64.
Preparing to unpack .../09-libctf0_2.34-6ubuntu1.4_amd64.deb ...
Unpacking libctf0:amd64 (2.34-6ubuntu1.4) ...
Selecting previously unselected package binutils-x86-64-linux-gnu.
Preparing to unpack .../10-binutils-x86-64-linux-gnu_2.34-6ubuntu1.4_amd64.deb ...
Unpacking binutils-x86-64-linux-gnu (2.34-6ubuntu1.4) ...
Selecting previously unselected package binutils.
Preparing to unpack .../11-binutils_2.34-6ubuntu1.4_amd64.deb ...
Unpacking binutils (2.34-6ubuntu1.4) ...
Selecting previously unselected package libgomp1:amd64.
Preparing to unpack .../12-libgomp1_10.3.0-1ubuntu1~20.04_amd64.deb ...
Unpacking libgomp1:amd64 (10.3.0-1ubuntu1~20.04) ...
Selecting previously unselected package libitm1:amd64.
Preparing to unpack .../13-libitm1_10.3.0-1ubuntu1~20.04_amd64.deb ...
Unpacking libitm1:amd64 (10.3.0-1ubuntu1~20.04) ...
Selecting previously unselected package libatomic1:amd64.
Preparing to unpack .../14-libatomic1_10.3.0-1ubuntu1~20.04_amd64.deb ...
Unpacking libatomic1:amd64 (10.3.0-1ubuntu1~20.04) ...
Selecting previously unselected package libasan5:amd64.
Preparing to unpack .../15-libasan5_9.4.0-1ubuntu1~20.04.1_amd64.deb ...
Unpacking libasan5:amd64 (9.4.0-1ubuntu1~20.04.1) ...
Selecting previously unselected package liblsan0:amd64.
Preparing to unpack .../16-liblsan0_10.3.0-1ubuntu1~20.04_amd64.deb ...
Unpacking liblsan0:amd64 (10.3.0-1ubuntu1~20.04) ...
Selecting previously unselected package libtsan0:amd64.
Preparing to unpack .../17-libtsan0_10.3.0-1ubuntu1~20.04_amd64.deb ...
Unpacking libtsan0:amd64 (10.3.0-1ubuntu1~20.04) ...
Selecting previously unselected package libubsan1:amd64.
Preparing to unpack .../18-libubsan1_10.3.0-1ubuntu1~20.04_amd64.deb ...
Unpacking libubsan1:amd64 (10.3.0-1ubuntu1~20.04) ...
Selecting previously unselected package libquadmath0:amd64.
Preparing to unpack .../19-libquadmath0_10.3.0-1ubuntu1~20.04_amd64.deb ...
Unpacking libquadmath0:amd64 (10.3.0-1ubuntu1~20.04) ...
Selecting previously unselected package libgcc-9-dev:amd64.
Preparing to unpack .../20-libgcc-9-dev_9.4.0-1ubuntu1~20.04.1_amd64.deb ...
Unpacking libgcc-9-dev:amd64 (9.4.0-1ubuntu1~20.04.1) ...
Selecting previously unselected package gcc-9.
Preparing to unpack .../21-gcc-9_9.4.0-1ubuntu1~20.04.1_amd64.deb ...
Unpacking gcc-9 (9.4.0-1ubuntu1~20.04.1) ...
Selecting previously unselected package gcc.
Preparing to unpack .../22-gcc_4%3a9.3.0-1ubuntu2_amd64.deb ...
Unpacking gcc (4:9.3.0-1ubuntu2) ...
Selecting previously unselected package libdpkg-perl.
Preparing to unpack .../23-libdpkg-perl_1.19.7ubuntu3.2_all.deb ...
Unpacking libdpkg-perl (1.19.7ubuntu3.2) ...
Selecting previously unselected package make.
Preparing to unpack .../24-make_4.2.1-1.2_amd64.deb ...
Unpacking make (4.2.1-1.2) ...
Selecting previously unselected package dpkg-dev.
Preparing to unpack .../25-dpkg-dev_1.19.7ubuntu3.2_all.deb ...
Unpacking dpkg-dev (1.19.7ubuntu3.2) ...
Selecting previously unselected package libc-dev-bin.
Preparing to unpack .../26-libc-dev-bin_2.31-0ubuntu9.9_amd64.deb ...
Unpacking libc-dev-bin (2.31-0ubuntu9.9) ...
Selecting previously unselected package linux-libc-dev:amd64.
Preparing to unpack .../27-linux-libc-dev_5.4.0-137.154_amd64.deb ...
Unpacking linux-libc-dev:amd64 (5.4.0-137.154) ...
Selecting previously unselected package libcrypt-dev:amd64.
Preparing to unpack .../28-libcrypt-dev_1%3a4.4.10-10ubuntu4_amd64.deb ...
Unpacking libcrypt-dev:amd64 (1:4.4.10-10ubuntu4) ...
Selecting previously unselected package libc6-dev:amd64.
Preparing to unpack .../29-libc6-dev_2.31-0ubuntu9.9_amd64.deb ...
Unpacking libc6-dev:amd64 (2.31-0ubuntu9.9) ...
Selecting previously unselected package libstdc++-9-dev:amd64.
Preparing to unpack .../30-libstdc++-9-dev_9.4.0-1ubuntu1~20.04.1_amd64.deb ...
Unpacking libstdc++-9-dev:amd64 (9.4.0-1ubuntu1~20.04.1) ...
Selecting previously unselected package g++-9.
Preparing to unpack .../31-g++-9_9.4.0-1ubuntu1~20.04.1_amd64.deb ...
Unpacking g++-9 (9.4.0-1ubuntu1~20.04.1) ...
Selecting previously unselected package g++.
Preparing to unpack .../32-g++_4%3a9.3.0-1ubuntu2_amd64.deb ...
Unpacking g++ (4:9.3.0-1ubuntu2) ...
Selecting previously unselected package build-essential.
Preparing to unpack .../33-build-essential_12.8ubuntu1.1_amd64.deb ...
Unpacking build-essential (12.8ubuntu1.1) ...
Selecting previously unselected package dctrl-tools.
Preparing to unpack .../34-dctrl-tools_2.24-3_amd64.deb ...
Unpacking dctrl-tools (2.24-3) ...
Selecting previously unselected package dkms.
Preparing to unpack .../35-dkms_2.8.1-5ubuntu2_all.deb ...
Unpacking dkms (2.8.1-5ubuntu2) ...
Selecting previously unselected package libfakeroot:amd64.
Preparing to unpack .../36-libfakeroot_1.24-1_amd64.deb ...
Unpacking libfakeroot:amd64 (1.24-1) ...
Selecting previously unselected package fakeroot.
Preparing to unpack .../37-fakeroot_1.24-1_amd64.deb ...
Unpacking fakeroot (1.24-1) ...
Selecting previously unselected package falco.
Preparing to unpack .../38-falco_0.33.1_amd64.deb ...
Unpacking falco (0.33.1) ...
Selecting previously unselected package libalgorithm-diff-perl.
Preparing to unpack .../39-libalgorithm-diff-perl_1.19.03-2_all.deb ...
Unpacking libalgorithm-diff-perl (1.19.03-2) ...
Selecting previously unselected package libalgorithm-diff-xs-perl.
Preparing to unpack .../40-libalgorithm-diff-xs-perl_0.04-6_amd64.deb ...
Unpacking libalgorithm-diff-xs-perl (0.04-6) ...
Selecting previously unselected package libalgorithm-merge-perl.
Preparing to unpack .../41-libalgorithm-merge-perl_0.08-3_all.deb ...
Unpacking libalgorithm-merge-perl (0.08-3) ...
Selecting previously unselected package libfile-fcntllock-perl.
Preparing to unpack .../42-libfile-fcntllock-perl_0.22-3build4_amd64.deb ...
Unpacking libfile-fcntllock-perl (0.22-3build4) ...
Selecting previously unselected package manpages-dev.
Preparing to unpack .../43-manpages-dev_5.05-1_all.deb ...
Unpacking manpages-dev (5.05-1) ...
Setting up manpages-dev (5.05-1) ...
Setting up libfile-fcntllock-perl (0.22-3build4) ...
Setting up libalgorithm-diff-perl (1.19.03-2) ...
Setting up binutils-common:amd64 (2.34-6ubuntu1.4) ...
Setting up linux-libc-dev:amd64 (5.4.0-137.154) ...
Setting up libctf-nobfd0:amd64 (2.34-6ubuntu1.4) ...
Setting up libgomp1:amd64 (10.3.0-1ubuntu1~20.04) ...
Setting up libfakeroot:amd64 (1.24-1) ...
Setting up fakeroot (1.24-1) ...
update-alternatives: using /usr/bin/fakeroot-sysv to provide /usr/bin/fakeroot (fakeroot) in auto mode
Setting up make (4.2.1-1.2) ...
Setting up libquadmath0:amd64 (10.3.0-1ubuntu1~20.04) ...
Setting up libmpc3:amd64 (1.1.0-1) ...
Setting up libatomic1:amd64 (10.3.0-1ubuntu1~20.04) ...
Setting up libdpkg-perl (1.19.7ubuntu3.2) ...
Setting up libubsan1:amd64 (10.3.0-1ubuntu1~20.04) ...
Setting up libcrypt-dev:amd64 (1:4.4.10-10ubuntu4) ...
Setting up libisl22:amd64 (0.22.1-1) ...
Setting up libbinutils:amd64 (2.34-6ubuntu1.4) ...
Setting up libc-dev-bin (2.31-0ubuntu9.9) ...
Setting up libalgorithm-diff-xs-perl (0.04-6) ...
Setting up libcc1-0:amd64 (10.3.0-1ubuntu1~20.04) ...
Setting up liblsan0:amd64 (10.3.0-1ubuntu1~20.04) ...
Setting up dctrl-tools (2.24-3) ...
Setting up libitm1:amd64 (10.3.0-1ubuntu1~20.04) ...
Setting up gcc-9-base:amd64 (9.4.0-1ubuntu1~20.04.1) ...
Setting up libalgorithm-merge-perl (0.08-3) ...
Setting up libtsan0:amd64 (10.3.0-1ubuntu1~20.04) ...
Setting up libctf0:amd64 (2.34-6ubuntu1.4) ...
Setting up libasan5:amd64 (9.4.0-1ubuntu1~20.04.1) ...
Setting up cpp-9 (9.4.0-1ubuntu1~20.04.1) ...
Setting up libc6-dev:amd64 (2.31-0ubuntu9.9) ...
Setting up binutils-x86-64-linux-gnu (2.34-6ubuntu1.4) ...
Setting up binutils (2.34-6ubuntu1.4) ...
Setting up dpkg-dev (1.19.7ubuntu3.2) ...
Setting up libgcc-9-dev:amd64 (9.4.0-1ubuntu1~20.04.1) ...
Setting up cpp (4:9.3.0-1ubuntu2) ...
Setting up gcc-9 (9.4.0-1ubuntu1~20.04.1) ...
Setting up libstdc++-9-dev:amd64 (9.4.0-1ubuntu1~20.04.1) ...
Setting up gcc (4:9.3.0-1ubuntu2) ...
Setting up dkms (2.8.1-5ubuntu2) ...
Setting up g++-9 (9.4.0-1ubuntu1~20.04.1) ...
Setting up g++ (4:9.3.0-1ubuntu2) ...
update-alternatives: using /usr/bin/g++ to provide /usr/bin/c++ (c++) in auto mode
Setting up build-essential (12.8ubuntu1.1) ...
Setting up falco (0.33.1) ...
Loading new falco-3.0.1+driver DKMS files...
Building for 5.15.0-1027-oracle
Building initial module for 5.15.0-1027-oracle
Can't load /var/lib/shim-signed/mok/.rnd into RNG
139937811137856:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:98:Filename=/var/lib/shim-signed/mok/.rnd
Generating a RSA private key
..........+++++
.......+++++
writing new private key to '/var/lib/shim-signed/mok/MOK.priv'
-----
This system doesn't support Secure Boot
Secure Boot not enabled on this system.
Done.

falco.ko:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/5.15.0-1027-oracle/updates/dkms/

depmod.....

DKMS: install completed.
Created symlink /etc/systemd/system/multi-user.target.wants/falco.service → /lib/systemd/system/falco.service.
Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
Processing triggers for man-db (2.9.1-1) ...
```

```
# systemctl status falco
● falco.service - Falco: Container Native Runtime Security
     Loaded: loaded (/lib/systemd/system/falco.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-01-23 13:04:33 UTC; 2min 9s ago
       Docs: https://falco.org/docs/
   Main PID: 249767 (falco)
      Tasks: 10 (limit: 19106)
     Memory: 18.9M
     CGroup: /system.slice/falco.service
             mq249767 /usr/bin/falco --pidfile=/var/run/falco.pid

Jan 23 13:04:33 worker-20230122 falco[249767]: Mon Jan 23 13:04:33 2023: Loading rules from file /etc/falco/falco_rules.local.yaml
Jan 23 13:04:34 worker-20230122 falco[249767]: The chosen syscall buffer dimension is: 8388608 bytes (8 MBs)
Jan 23 13:04:34 worker-20230122 falco[249767]: Mon Jan 23 13:04:34 2023: The chosen syscall buffer dimension is: 8388608 bytes (8 MBs)
Jan 23 13:04:34 worker-20230122 falco[249767]: Mon Jan 23 13:04:34 2023: Starting health webserver with threadiness 2, listening on port 8765
Jan 23 13:04:34 worker-20230122 falco[249767]: Starting health webserver with threadiness 2, listening on port 8765
Jan 23 13:04:34 worker-20230122 falco[249767]: Enabled event sources: syscall
Jan 23 13:04:34 worker-20230122 falco[249767]: Mon Jan 23 13:04:34 2023: Enabled event sources: syscall
Jan 23 13:04:34 worker-20230122 falco[249767]: Mon Jan 23 13:04:34 2023: Opening capture with Kernel module
Jan 23 13:04:34 worker-20230122 falco[249767]: Opening capture with Kernel module
Jan 23 13:04:35 worker-20230122 falco[249767]: 13:04:35.642447687: Error File below /etc opened for writing (user=root user_loginuid=-1 command=ruby /opt/unified-monitoring-agent/embedded/bin/fluent_config_updater.rb -c /etc/unified-mon>
```

設定ファイルを確認する。

```
# cd /etc/falco/
# ls
aws_cloudtrail_rules.yaml  falco.yaml  falco_rules.local.yaml  falco_rules.yaml  k8s_audit_rules.yaml  rules.available  rules.d
```
### 動作確認

ログファイルを確認する。

Podを作成してみる。

```
# tail -f /var/log/syslog | grep falco
Jan 23 13:11:45 worker-20230122 falco: 13:11:45.642601230: Warning a shell configuration file has been modified (user=root user_loginuid=-1 command=containerd pid=831 pcmdline=systemd file=/var/lib/containerd/tmpmounts/containerd-mount42069301/etc/skel/.bash_logout container_id=host image=<NA>)
Jan 23 13:11:45 worker-20230122 falco: 13:11:45.642689096: Warning a shell configuration file has been modified (user=root user_loginuid=-1 command=containerd pid=831 pcmdline=systemd file=/var/lib/containerd/tmpmounts/containerd-mount42069301/etc/skel/.bashrc container_id=host image=<NA>)
Jan 23 13:11:45 worker-20230122 falco: 13:11:45.642774226: Warning a shell configuration file has been modified (user=root user_loginuid=-1 command=containerd pid=831 pcmdline=systemd file=/var/lib/containerd/tmpmounts/containerd-mount42069301/etc/skel/.profile container_id=host image=<NA>)
Jan 23 13:11:45 worker-20230122 falco: 13:11:45.825957071: Warning a shell configuration file has been modified (user=root user_loginuid=-1 command=containerd pid=831 pcmdline=systemd file=/var/lib/containerd/tmpmounts/containerd-mount42069301/root/.bashrc container_id=host image=<NA>)
Jan 23 13:11:45 worker-20230122 falco: 13:11:45.826020671: Warning a shell configuration file has been modified (user=root user_loginuid=-1 command=containerd pid=831 pcmdline=systemd file=/var/lib/containerd/tmpmounts/containerd-mount42069301/root/.profile container_id=host image=<NA>)
Jan 23 13:11:47 worker-20230122 falco: 13:11:47.956966905: Warning Log files were tampered (user=root user_loginuid=-1 command=containerd pid=831 file=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/101/fs/var/log/dpkg.log container_id=host image=<NA>)
```

Master nodeでコンテナにログインする。

```
# tail -f /var/log/syslog | grep falco
Jan 23 13:19:06 worker-20230122 falco: 13:19:06.607870627: Notice A shell was spawned in a container with an attached terminal (user=<NA> user_loginuid=-1 nginx (id=4edbcbca7577) shell=bash parent=runc cmdline=bash pid=263941 terminal=34816 container_id=4edbcbca7577 image=docker.io/library/nginx)
```

コンテナの`/etc/passwd`に追記する。

```
Jan 23 13:21:22 worker-20230122 falco: 13:21:22.193504721: Error File below /etc opened for writing (user=<NA> user_loginuid=-1 command=bash pid=263941 parent=<NA> pcmdline=<NA> file=/etc/passwd program=bash gparent=<NA> ggparent=<NA> gggparent=<NA> container_id=4edbcbca7577 image=docker.io/library/nginx)
```

コンテナ内で`apt-get update`する。

```
Jan 23 13:23:23 worker-20230122 falco: 13:23:23.442834341: Error Package management process launched in container (user=<NA> user_loginuid=-1 command=apt-get update pid=266578 container_id=4edbcbca7577 container_name=nginx image=docker.io/library/nginx:latest)
```

コンテナからログアウトする

```
Jan 23 13:24:31 worker-20230122 falco: 13:24:31.858405923: Warning Shell history had been deleted or renamed (user=<NA> user_loginuid=-1 type=openat command=bash pid=263941 fd.name=/root/.bash_history name=/root/.bash_history path=<NA> oldpath=<NA> nginx (id=4edbcbca7577))
```