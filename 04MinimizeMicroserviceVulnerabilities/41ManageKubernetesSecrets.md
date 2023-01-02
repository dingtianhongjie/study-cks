# マニュアル
https://kubernetes.io/ja/docs/concepts/configuration/secret/

# 学習ログ
## Secretの作成と利用
Secretを２つ作成する。

```
$ kubectl create secret generic secret1 --from-literal user=admin
secret/secret1 created
$ kubectl create secret generic secret2 --from-literal pass=1234
secret/secret2 created
$ kubectl get secrets
NAME      TYPE     DATA   AGE
secret1   Opaque   1      22s
secret2   Opaque   1      6s
```

この２つのSecretを使用するPodを作成する。
secret1はVolumeとしてマウント、secret2は環境変数として使用する。

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
  - image: nginx
    name: pod
    resources: {}
    env:
      - name: PASSWORD
        valueFrom:
          secretKeyRef:
            name: secret2
            key: pass
    volumeMounts:
    - name: secret1
      mountPath: "/etc/secret1"
      readOnly: true
  volumes:
  - name: secret1
    secret:
      secretName: secret1
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
$ kubectl apply -f pod.yaml
pod/pod created
$ kubectl get pod
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          13s
```

確認する。

```
$ kubectl exec pod -- env |grep PASS
PASSWORD=1234
$ kubectl exec pod -- mount |grep secret1
tmpfs on /etc/secret1 type tmpfs (ro,relatime,size=16278444k,inode64)
$ kubectl exec pod -- ls /etc/secret1
user
$ kubectl exec pod -- cat /etc/secret1/user
admin
```

## Worker nodeからSecretを確認してみる（セキュリティの問題）

Podがデプロイされているノードを確認する。
→Workerノード
```
$ kubectl get pod -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP               NODE             NOMINATED NODE   READINESS GATES
pod    1/1     Running   0          22m   192.168.119.32   worker20221221   <none>           <none>
```

Workerノードで稼働しているコンテナを確認する。

```
worker# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
59e482a698a1d       1403e55ab369c       23 minutes ago      Running             pod                       0                   4505fa711b2b4       pod
a5c5ee258e9ee       38b76de417d5d       6 hours ago         Running             calico-kube-controllers   6                   ef2deb77a9abe       calico-kube-controllers-798cc86c47-twnmm
808dbef7d7cef       5185b96f0becf       6 hours ago         Running             coredns                   6                   8760a23630c3d       coredns-565d847f94-qjjk9
58a547e5156c7       5185b96f0becf       6 hours ago         Running             coredns                   6                   084dcf17434a8       coredns-565d847f94-bkpdp
8deb531130114       54637cb36d4a1       6 hours ago         Running             calico-node               6                   929424f7f47af       calico-node-lshdc
2beb3f2d521b2       430bf372a28b7       6 hours ago         Running             kube-proxy                6                   8dda991334313       kube-proxy-m7zmd
```

`pod`のContainer IDを指定して、コンテナの詳細を確認する。

```
worker# crictl inspect 59e482a698a1d
{
  "status": {
    "id": "59e482a698a1dd3a1712af176f050926fd5c1ea851ba23a08a6c931db773dc85",
    "metadata": {
      "attempt": 0,
      "name": "pod"
    },
    "state": "CONTAINER_RUNNING",
    "createdAt": "2023-01-02T12:49:55.946286349Z",
    "startedAt": "2023-01-02T12:49:56.011352197Z",
    "finishedAt": "0001-01-01T00:00:00Z",
    "exitCode": 0,
    "image": {
      "annotations": {},
      "image": "docker.io/library/nginx:latest"
・・・
    "mounts": [
      {
        "containerPath": "/etc/secret1",
        "hostPath": "/var/lib/kubelet/pods/d5361bb9-252e-4e4b-a5b7-54be27fe7ca7/volumes/kubernetes.io~secret/secret1",
        "propagation": "PROPAGATION_PRIVATE",
        "readonly": true,
        "selinuxRelabel": false
      },
・・・
      "envs": [
        {
          "key": "PASSWORD",
          "value": "1234"
・・・
```

Workerノードにrootでログインできると、環境変数で指定したSecretの内容は見えるが、VolumeでマウントしたSecretの値はわからない。

コンテナのpidを確認する。

```
worker# crictl inspect 59e482a698a1d |grep pid
    "pid": 206898,
            "pid": 1
            "type": "pid"
```

pidを指定して、コンテナのrootファイルシステムを確認できる。

```
worker# ls /proc/206898/root
bin  boot  dev  docker-entrypoint.d  docker-entrypoint.sh  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

この方法ならVolumeマウントしたSecretの値も確認できてしまう。

```
# cat /proc/206898/root/etc/secret1/user
admin
```