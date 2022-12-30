# マニュアル
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

# 学習ログ
## Security Context
### uid / gid

以下のマニフェストのPodを作成する。

```pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
$ kubectl apply -f pod.yaml
pod/pod created
$ kubectl get pod
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          12s
```

コンテナにログインして、権限を確認する。

```
$ kubectl exec -it pod -- sh
/ # id
uid=0(root) gid=0(root) groups=10(wheel)
/ # touch test
/ # ls -l test
-rw-r--r--    1 root     root             0 Dec 30 06:32 test
```

マニフェストを以下のように書き換えて、SecurityContextを設定する。

```pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Podを入れ替える。

```
$ kubectl replace -f pod.yaml --force
pod "pod" deleted
pod/pod replaced
$ kubectl get pod
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          14s
```

コンテナにログインして、権限を確認する。
uidが1000なので、/直下には書き込みができない。

```
$ kubectl exec -it pod -- sh
/ $ id
uid=1000 gid=3000
/ $ touch test
touch: test: Permission denied
/ $
```
/tmp配下だったら書き込みができる。

```
/ $ cd /tmp/
/tmp $ touch test
/tmp $ ls -l
total 0
-rw-r--r--    1 1000     3000             0 Dec 30 06:38 test
```

### non root

コンテナにSecurityContextを追加して、non rootでコンテナを起動させる。

```pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
    securityContext:
      runAsNonRoot: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Podを入れ替える。

```
$ kubectl replace -f pod.yaml --force
pod "pod" deleted
pod/pod replaced
$ kubectl get pod
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          27s
```