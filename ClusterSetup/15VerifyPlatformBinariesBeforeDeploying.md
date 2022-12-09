# マニュアル
https://github.com/kubernetes/kubernetes

# 学習ログ
## Kubernetesのバイナリのハッシュ値を確認する。

Kubernetesのバージョンを確認する。
```
$ kubectl get node
NAME              STATUS   ROLES                  AGE   VERSION
master-20221124   Ready    control-plane,master   4d    v1.23.3
worker-20221124   Ready    <none>                 4d    v1.23.3
```

- `Github → tags` で自分がインストールしているKubernetesのバージョンのリンクをクリック
    - 今回はv1.23.3だが、リンクが一覧にないので、URLをそのバージョンに書き換える。
- `CHANGELOG`をクリック
- Server Binariesの`kubernetes-server-linux-amd64.tar.gz`のLinkをコピーする。
- Master nodeでバイナリをダウンロードする。

```
$ wget https://dl.k8s.io/v1.23.4/kubernetes-server-linux-amd64.tar.gz
--2022-11-28 13:10:50--  https://dl.k8s.io/v1.23.4/kubernetes-server-linux-amd64.tar.gz
Resolving dl.k8s.io (dl.k8s.io)... 34.107.204.206, 2600:1901:0:26f3::
Connecting to dl.k8s.io (dl.k8s.io)|34.107.204.206|:443... connected.
HTTP request sent, awaiting response... 302 Moved Temporarily
Location: https://storage.googleapis.com/kubernetes-release/release/v1.23.4/kubernetes-server-linux-amd64.tar.gz [following]
--2022-11-28 13:10:50--  https://storage.googleapis.com/kubernetes-release/release/v1.23.4/kubernetes-server-linux-amd64.tar.gz
Resolving storage.googleapis.com (storage.googleapis.com)... 142.251.163.128, 142.251.16.128, 142.250.188.208, ...
Connecting to storage.googleapis.com (storage.googleapis.com)|142.251.163.128|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 341392468 (326M) [application/x-tar]
Saving to: ‘kubernetes-server-linux-amd64.tar.gz’

kubernetes-server-linux-amd64.tar.gz                        100%[========================================================================================================================================>] 325.58M   120MB/s    in 2.7s

2022-11-28 13:10:53 (120 MB/s) - ‘kubernetes-server-linux-amd64.tar.gz’ saved [341392468/341392468]
```

ダウンロードしたバイナリのハッシュ値を確認する。

```
$ sha512sum kubernetes-server-linux-amd64.tar.gz
cd5e90d25bd48dbbd1755eb5d328676c6424d191c2936d7477c9dde72cd374389f888bfee6ffd02836bbe136e8bf9a2cb8adcad2c650714dc6112075949c06c7  kubernetes-server-linux-amd64.tar.gz
```

確認したハッシュ値とGithub上に記載されているハッシュ値が同じであることを確認する。

## APIサーバのハッシュ値の確認
### バイナリのハッシュ値

ダウンロードしたバイナリを展開する。

```
$ tar xzf kubernetes-server-linux-amd64.tar.gz
$ ls
kubernetes  kubernetes-server-linux-amd64.tar.gz
```

API Serverのバージョンを確認する。

```
$ kubernetes/server/bin/kube-apiserver --version
Kubernetes v1.23.4
```

実行されているAPI Serverのバージョンを確認する。
```
$ kubectl -n kube-system get pod | grep api
kube-apiserver-master-20221124             1/1     Running   2 (43m ago)   4d
$ kubectl -n kube-system get pod kube-apiserver-master-20221124 -o yaml | grep image
    image: k8s.gcr.io/kube-apiserver:v1.23.14
    imagePullPolicy: IfNotPresent
    image: k8s.gcr.io/kube-apiserver:v1.23.14
    imageID: docker-pullable://k8s.gcr.io/kube-apiserver@sha256:d5b350f940b196cd4134c35f6fb375e0c13d96b4a316bd8633dcb19160280f5e
```
あれ？API Serverはv1.23.14になってる。

仕方ないので、同様にv1.23.14のバイナリをダウンロードして展開する。

```
$ wget https://dl.k8s.io/v1.23.14/kubernetes-server-linux-amd64.tar.gz
--2022-11-28 13:40:47--  https://dl.k8s.io/v1.23.14/kubernetes-server-linux-amd64.tar.gz
Resolving dl.k8s.io (dl.k8s.io)... 34.107.204.206, 2600:1901:0:26f3::
Connecting to dl.k8s.io (dl.k8s.io)|34.107.204.206|:443... connected.
HTTP request sent, awaiting response... 302 Moved Temporarily
Location: https://storage.googleapis.com/kubernetes-release/release/v1.23.14/kubernetes-server-linux-amd64.tar.gz [following]
--2022-11-28 13:40:47--  https://storage.googleapis.com/kubernetes-release/release/v1.23.14/kubernetes-server-linux-amd64.tar.gz
Resolving storage.googleapis.com (storage.googleapis.com)... 142.251.16.128, 142.250.65.80, 172.217.15.112, ...
Connecting to storage.googleapis.com (storage.googleapis.com)|142.251.16.128|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 341994927 (326M) [application/x-tar]
Saving to: ‘kubernetes-server-linux-amd64.tar.gz’

kubernetes-server-linux-amd64.tar.gz                        100%[========================================================================================================================================>] 326.15M   135MB/s    in 2.4s

2022-11-28 13:40:50 (135 MB/s) - ‘kubernetes-server-linux-amd64.tar.gz’ saved [341994927/341994927]
$ tar xzf kubernetes-server-linux-amd64.tar.gz
$ kubernetes/server/bin/kube-apiserver --version
Kubernetes v1.23.14
```

バイナリのハッシュ値をメモしておく。
```
$ sha512sum kubernetes/server/bin/kube-apiserver > api-hash
```

### API Serverのコンテナ上のハッシュ値
kube-apiserverのプロセスIDを確認する。

```
$ ps aux | grep kube-api
root        2527  3.3  1.8 1043132 308520 ?      Ssl  12:44   2:00 kube-apiserver --advertise-address=10.0.0.71 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
ubuntu     47646  0.0  0.0   8168  2380 pts/1    R+   13:45   0:00 grep --color=auto kube-api
```

API Serverのrootディレクトリを確認する。
```
$ sudo ls /proc/2527/root
bin  boot  dev  etc  go-runner  home  lib  proc  root  run  sbin  sys  tmp  usr  var
$ sudo find /proc/2527/root/ -name kube-apiserver
/proc/2527/root/usr/local/bin/kube-apiserver
```

コンテナ上のハッシュ値を確認する。
```
$ sudo sha512sum /proc/2527/root/usr/local/bin/kube-apiserver >> api-hash
```

バイナリのハッシュ値とコンテナ上のハッシュ値が同じであることを確認する。
```
$ cat api-hash
8abaebc26a1b964de6be139eea98fc6799f74b7798a58012bf02e54f5d1ceff668b55630844e3d43a7e510017a804299f0edd06c5891aa34ce09c0850701f051  kubernetes/server/bin/kube-apiserver
8abaebc26a1b964de6be139eea98fc6799f74b7798a58012bf02e54f5d1ceff668b55630844e3d43a7e510017a804299f0edd06c5891aa34ce09c0850701f051  /proc/2527/root/usr/local/bin/kube-apiserver
```
