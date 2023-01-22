# マニュアル

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