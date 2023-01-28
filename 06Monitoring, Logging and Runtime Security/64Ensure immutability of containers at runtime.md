# マニュアル
https://kubernetes.io/ja/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
https://kubernetes.io/docs/concepts/storage/volumes/

# 学習ログ
## コマンド、Shellの削除

StartupProbeを使用して、コンテナからtouchコマンドを削除して使えないようにする。

```pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: immutable
  name: immutable
spec:
  containers:
  - image: httpd
    name: immutable
    resources: {}
    startupProbe:
      exec:
        command:
        - rm
        - /bin/touch
      initialDelaySeconds: 1
      periodSeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
$ kubectl apply -f pod.yaml
pod/immutable created
$ kubectl get pod
NAME        READY   STATUS    RESTARTS   AGE
immutable   1/1     Running   0          10s
```

```
$ kubectl exec -it immutable -- bash
# touch test
bash: touch: command not found
```

touchではなく、`/bin/bash`を削除すれば、Bashでログインできなくなる。

## Read Only Filesystem
SecurityContextでRoot FilesystemをReadOnlyにする。

```pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: immutable
  name: immutable
spec:
  containers:
  - image: httpd
    name: immutable
    resources: {}
    securityContext:
      readOnlyRootFilesystem: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
$ kubectl apply -f pod.yaml
pod/immutable created
$ kubectl get pod
NAME        READY   STATUS             RESTARTS     AGE
immutable   0/1     CrashLoopBackOff   1 (4s ago)   5s
```

Podが起動しない。
ログを確認する。

```
$ kubectl logs immutable
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 192.168.126.198. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 192.168.126.198. Set the 'ServerName' directive globally to suppress this message
[Sat Jan 28 13:29:34.755862 2023] [core:error] [pid 1:tid 139943238802752] (30)Read-only file system: AH00099: could not create /usr/local/apache2/logs/httpd.pid.adFPwh
[Sat Jan 28 13:29:34.755930 2023] [core:error] [pid 1:tid 139943238802752] AH00100: httpd: could not log pid to file /usr/local/apache2/logs/httpd.pid
```

Apacheのログファイルが作れないので、エラーになっている。

これを回避するために、emptyDirで`/usr/local/apache2/logs`にマウントする。

```pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: immutable
  name: immutable
spec:
  containers:
  - image: httpd
    name: immutable
    resources: {}
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
    - mountPath: /usr/local/apache2/logs
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
$ kubectl delete -f pod.yaml
pod "immutable" deleted
$ kubectl apply -f pod.yaml
pod/immutable created
```

```
$ kubectl get pod
NAME        READY   STATUS    RESTARTS   AGE
immutable   1/1     Running   0          19s
```

ReadOnlyなので、書き込みはできない。

```
$ kubectl exec -it immutable -- touch test
touch: cannot touch 'test': Read-only file system
command terminated with exit code 1
```

emptyDirをマウントしたので、ログファイルはある。

```
$ kubectl exec -it immutable -- ls /usr/local/apache2/logs/
httpd.pid
```

ここには書き込みができる。

```
$ kubectl exec -it immutable -- touch /usr/local/apache2/logs/test
$ kubectl exec -it immutable -- ls /usr/local/apache2/logs/
httpd.pid  test
```