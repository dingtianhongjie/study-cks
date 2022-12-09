# マニュアル
https://kubernetes.io/docs/concepts/security/controlling-access/

#  学習ログ
## Anonymous user でのアクセス

API Serverに自己証明書をacceptしてアクセスする。
```
$ curl https://localhost:6443 -k
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {},
  "code": 403
```

Forbiddenだけど、システム上禁止されているということなので、Anonymousユーザからのリクエストは受け付けている。

API Serverのマニフェストを編集し、Anonymousユーザからのアクセスを禁止する。

```
$sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml
```

```kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.0.0.71:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=10.0.0.71
    - --anonymous-auth=false #追記
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
```

API Serverが再起動する。
※ちなみに、Anonymousユーザを拒否すると、Liveness ProbeによってAPI Serverは再起動を繰り返す。

```
$ kubectl -n kube-system get pod |grep api
kube-apiserver-master-20221124             0/1     Running   0             63s
```

アクセスが拒否されることを確認する。

```
$ curl https://localhost:6443 -k
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "Unauthorized",
  "reason": "Unauthorized",
  "code": 401
```

API Serverのマニフェストを元に戻しておく。

## Manual API request

kube config fileを確認する。

```
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.0.0.71:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

Credential情報がマスクされている。
--rawオプションをつけると、Credential情報も表示される。

```
$ kubectl config view --raw
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    server: https://10.0.0.71:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    client-key-data: LS0tLS1CRUdJTiBSU0xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

これは`.kube/config`ファイルと同じ内容

```
$ cat ~/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0t
・・・
```

Credential情報はEncodeされているので、Decodeすることで確認できる。
ClusterのCredential情報をDecodeする。

```
$ echo LS0tLS1CRUdJTiBDRVJUSxxxxxxxxxxxxxxx== | base64 -d
-----BEGIN CERTIFICATE-----
MIIC/jCCAeagAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJl
・・・
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
・・・
Cn4=
-----END CERTIFICATE-----
```

これをファイルにはき出す

```
$ echo LS0tLS1CRUdJTiBDRVJUSxxxxxxxxxxxxxxx== | base64 -d > ca
```
同じ方法でclient-certificate-dataとclient-key-dataをDecodeしてはき出す。

```
$ ls -l
total 12
-rw-rw-r-- 1 ubuntu ubuntu 1099 Nov 30 13:00 ca
-rw-rw-r-- 1 ubuntu ubuntu 1147 Nov 30 13:03 crt
-rw-rw-r-- 1 ubuntu ubuntu 1679 Nov 30 13:03 key
```

### API Serverへのアクセス
config viewで確認したAPI Sererにcurlでアクセスする。

```
$ curl https://10.0.0.71:6443
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

アクセスできるけど、認証ができない。

--cacertでClusterのCredential情報を指定してアクセスする。

```
$ curl https://10.0.0.71:6443 --cacert ca
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {},
  "code": 403
```

認証されたけど、anonymous userになっている。

さらにClientとClient keyを指定してアクセスする。

```
$ curl https://10.0.0.71:6443 --cacert ca --cert crt --key key
{
  "paths": [
    "/.well-known/openid-configuration",
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1",
    "/apis/apiregistration.k8s.io",
    "/apis/apiregistration.k8s.io/v1",
    "/apis/apps",
    "/apis/apps/v1",
    "/apis/authentication.k8s.io",
    "/apis/authentication.k8s.io/v1",
    "/apis/authorization.k8s.io",
    "/apis/authorization.k8s.io/v1",
    "/apis/autoscaling",
    "/apis/autoscaling/v1",
    "/apis/autoscaling/v2",
    "/apis/autoscaling/v2beta1",
    "/apis/autoscaling/v2beta2",
    "/apis/batch",
    "/apis/batch/v1",
    "/apis/batch/v1beta1",
    "/apis/certificates.k8s.io",
    "/apis/certificates.k8s.io/v1",
    "/apis/coordination.k8s.io",
    "/apis/coordination.k8s.io/v1",
    "/apis/crd.projectcalico.org",
    "/apis/crd.projectcalico.org/v1",
    "/apis/discovery.k8s.io",
    "/apis/discovery.k8s.io/v1",
    "/apis/discovery.k8s.io/v1beta1",
    "/apis/events.k8s.io",
    "/apis/events.k8s.io/v1",
    "/apis/events.k8s.io/v1beta1",
    "/apis/flowcontrol.apiserver.k8s.io",
    "/apis/flowcontrol.apiserver.k8s.io/v1beta1",
    "/apis/flowcontrol.apiserver.k8s.io/v1beta2",
    "/apis/networking.k8s.io",
    "/apis/networking.k8s.io/v1",
    "/apis/node.k8s.io",
    "/apis/node.k8s.io/v1",
    "/apis/node.k8s.io/v1beta1",
    "/apis/policy",
    "/apis/policy/v1",
    "/apis/policy/v1beta1",
    "/apis/rbac.authorization.k8s.io",
    "/apis/rbac.authorization.k8s.io/v1",
    "/apis/scheduling.k8s.io",
    "/apis/scheduling.k8s.io/v1",
    "/apis/storage.k8s.io",
    "/apis/storage.k8s.io/v1",
    "/apis/storage.k8s.io/v1beta1",
    "/healthz",
    "/healthz/autoregister-completion",
    "/healthz/etcd",
    "/healthz/log",
    "/healthz/ping",
    "/healthz/poststarthook/aggregator-reload-proxy-client-cert",
    "/healthz/poststarthook/apiservice-openapi-controller",
    "/healthz/poststarthook/apiservice-registration-controller",
    "/healthz/poststarthook/apiservice-status-available-controller",
    "/healthz/poststarthook/bootstrap-controller",
    "/healthz/poststarthook/crd-informer-synced",
    "/healthz/poststarthook/generic-apiserver-start-informers",
    "/healthz/poststarthook/kube-apiserver-autoregistration",
    "/healthz/poststarthook/priority-and-fairness-config-consumer",
    "/healthz/poststarthook/priority-and-fairness-config-producer",
    "/healthz/poststarthook/priority-and-fairness-filter",
    "/healthz/poststarthook/rbac/bootstrap-roles",
    "/healthz/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/healthz/poststarthook/start-apiextensions-controllers",
    "/healthz/poststarthook/start-apiextensions-informers",
    "/healthz/poststarthook/start-cluster-authentication-info-controller",
    "/healthz/poststarthook/start-kube-aggregator-informers",
    "/healthz/poststarthook/start-kube-apiserver-admission-initializer",
    "/livez",
    "/livez/autoregister-completion",
    "/livez/etcd",
    "/livez/log",
    "/livez/ping",
    "/livez/poststarthook/aggregator-reload-proxy-client-cert",
    "/livez/poststarthook/apiservice-openapi-controller",
    "/livez/poststarthook/apiservice-registration-controller",
    "/livez/poststarthook/apiservice-status-available-controller",
    "/livez/poststarthook/bootstrap-controller",
    "/livez/poststarthook/crd-informer-synced",
    "/livez/poststarthook/generic-apiserver-start-informers",
    "/livez/poststarthook/kube-apiserver-autoregistration",
    "/livez/poststarthook/priority-and-fairness-config-consumer",
    "/livez/poststarthook/priority-and-fairness-config-producer",
    "/livez/poststarthook/priority-and-fairness-filter",
    "/livez/poststarthook/rbac/bootstrap-roles",
    "/livez/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/livez/poststarthook/start-apiextensions-controllers",
    "/livez/poststarthook/start-apiextensions-informers",
    "/livez/poststarthook/start-cluster-authentication-info-controller",
    "/livez/poststarthook/start-kube-aggregator-informers",
    "/livez/poststarthook/start-kube-apiserver-admission-initializer",
    "/logs",
    "/metrics",
    "/openapi/v2",
    "/openid/v1/jwks",
    "/readyz",
    "/readyz/autoregister-completion",
    "/readyz/etcd",
    "/readyz/informer-sync",
    "/readyz/log",
    "/readyz/ping",
    "/readyz/poststarthook/aggregator-reload-proxy-client-cert",
    "/readyz/poststarthook/apiservice-openapi-controller",
    "/readyz/poststarthook/apiservice-registration-controller",
    "/readyz/poststarthook/apiservice-status-available-controller",
    "/readyz/poststarthook/bootstrap-controller",
    "/readyz/poststarthook/crd-informer-synced",
    "/readyz/poststarthook/generic-apiserver-start-informers",
    "/readyz/poststarthook/kube-apiserver-autoregistration",
    "/readyz/poststarthook/priority-and-fairness-config-consumer",
    "/readyz/poststarthook/priority-and-fairness-config-producer",
    "/readyz/poststarthook/priority-and-fairness-filter",
    "/readyz/poststarthook/rbac/bootstrap-roles",
    "/readyz/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/readyz/poststarthook/start-apiextensions-controllers",
    "/readyz/poststarthook/start-apiextensions-informers",
    "/readyz/poststarthook/start-cluster-authentication-info-controller",
    "/readyz/poststarthook/start-kube-aggregator-informers",
    "/readyz/poststarthook/start-kube-apiserver-admission-initializer",
    "/readyz/shutdown",
    "/version"
  ]
}
```

## 外部からAPI Serverへのアクセス

外部からアクセスするため、NodePortを作成する。
ここでは、デフォルトのClusterIPをNodePortに変える。

```
$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   7d
$ kubectl edit svc kubernetes
service/kubernetes edited
$ kubectl get svc
NAME         TYPE       CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
kubernetes   NodePort   10.96.0.1    <none>        443:32073/TCP   7d
```

外部からAPI Serverにアクセスできることを確認する。

```
external$ curl https://143.47.99.33:32073
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
external$ curl https://143.47.99.33:32073 -k
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {},
  "code": 403
}
```

Master NodeのKube Configファイルを外部サーバにコピーする。

```
$ kubectl config view --raw
apiVersion: v1
clusters:
- cluster:
・・・
```

```
external$ ls -l
total 8
-rw-rw-r-- 1 ubuntu ubuntu 5637 Dec  1 12:58 conf
```

コピーしたConfigファイルを指定して、kubectlコマンドを実行する。

```
external$ kubectl --kubeconfig conf get node
^C
```
API Serverにアクセスできない。
ConfファイルのserverがローカルIPなので、アクセスできない。そこで、ConfファイルのserverをExternal IPアドレスに書き換える。

```conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx==
    server: https://143.47.99.xxx:32073 # Change
  name: kubernetes
・・・
```

再度API Serverにアクセスする。

```
external$ kubectl --kubeconfig conf get node
Unable to connect to the server: x509: certificate is valid for 10.96.0.1, 10.0.0.71, not 143.47.99.xxx
```

API Serverにはアクセスできるが、外部サーバ(143.47.99.xxx)の証明書がないためコマンドは失敗する。

Master ServerのPrivate Keyを確認する。

```
$ cd /etc/kubernetes/pki/
$ ls -l
total 60
-rw-r--r-- 1 root root 1155 Nov 24 12:40 apiserver-etcd-client.crt
-rw------- 1 root root 1675 Nov 24 12:40 apiserver-etcd-client.key
-rw-r--r-- 1 root root 1164 Nov 24 12:40 apiserver-kubelet-client.crt
-rw------- 1 root root 1679 Nov 24 12:40 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1294 Nov 24 12:40 apiserver.crt
-rw------- 1 root root 1675 Nov 24 12:40 apiserver.key
-rw-r--r-- 1 root root 1099 Nov 24 12:40 ca.crt
-rw------- 1 root root 1675 Nov 24 12:40 ca.key
drwxr-xr-x 2 root root 4096 Nov 24 12:40 etcd
-rw-r--r-- 1 root root 1115 Nov 24 12:40 front-proxy-ca.crt
-rw------- 1 root root 1679 Nov 24 12:40 front-proxy-ca.key
-rw-r--r-- 1 root root 1119 Nov 24 12:40 front-proxy-client.crt
-rw------- 1 root root 1675 Nov 24 12:40 front-proxy-client.key
-rw------- 1 root root 1675 Nov 24 12:40 sa.key
-rw------- 1 root root  451 Nov 24 12:40 sa.pub
```

`apiserver.crt`ファイルを確認する。

```
$ openssl x509 -in apiserver.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 4345368812112292906 (0x3c4dd9e97003ac2a)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Nov 24 12:40:05 2022 GMT
            Not After : Nov 24 12:40:05 2023 GMT
        Subject: CN = kube-apiserver
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:9e:fa:79:d0:38:8e:2e:70:0b:aa:23:47:d3:a1:
                    ed:dc:72:6d:52:e4:ad:ec:34:35:01:cc:62:b8:50:
                    51:f5:03:04:25:21:a3:3b:13:16:3c:fe:41:f3:8b:
                    db:c3:0f:9d:ce:4c:6e:e3:46:29:74:b0:55:b4:54:
                    f8:6e:4b:43:48:4b:c5:ca:a8:d7:03:f8:66:9f:d9:
                    04:a5:50:7c:37:fb:25:2b:43:eb:b8:43:b1:22:60:
                    18:6e:26:10:ca:4f:7f:15:5d:0f:d5:21:4e:42:e3:
                    5b:25:53:8d:ef:52:35:e8:9e:2b:28:23:bc:5d:33:
                    90:1e:3d:e6:e6:d5:67:81:58:07:af:a2:b8:e2:9e:
                    7f:b5:5b:50:59:43:94:e1:0d:aa:f7:a8:51:cf:08:
                    1f:64:04:a9:94:62:8e:f5:93:ff:72:7c:b7:64:ef:
                    4b:4a:cf:e0:97:6e:07:b4:cb:79:94:66:2b:2e:32:
                    26:30:94:d4:48:f5:a6:a3:3e:94:2e:38:a8:6b:13:
                    e6:aa:33:04:7d:80:18:26:71:ed:cd:d1:61:df:d0:
                    87:19:ea:a8:e8:69:0d:e6:5e:d4:80:30:98:20:3b:
                    83:8f:e4:c3:09:fc:a3:7c:92:7a:19:9e:45:cd:14:
                    b2:cf:8f:3c:9a:c6:4a:5b:6d:1a:45:0b:ab:25:d6:
                    10:fd
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier:
                keyid:A6:F7:2F:A2:25:0C:A8:F5:D8:7D:33:34:ED:31:1D:42:D4:CF:1E:A2

            X509v3 Subject Alternative Name:
                DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, DNS:master-20221124, IP Address:10.96.0.1, IP Address:10.0.0.71
    Signature Algorithm: sha256WithRSAEncryption
         0e:b7:61:da:40:7c:4e:01:72:15:23:1e:de:df:0f:31:d7:01:
         4f:06:77:d5:42:99:7d:0a:f8:20:60:fd:a7:77:d2:ec:44:2d:
         d9:f1:6c:34:51:9e:0a:84:ef:5d:54:e4:c5:45:da:06:56:96:
         0c:a7:0d:09:0e:be:51:f6:43:78:a7:75:ca:86:47:27:48:4d:
         73:27:8f:3c:b0:8d:b9:5c:24:b4:4f:25:72:63:8e:03:04:2d:
         1e:99:30:5e:db:97:87:0b:ca:3f:8d:33:e5:fc:cb:26:a3:d4:
         18:33:cb:55:5f:cd:3c:cd:bc:3f:b9:d6:9c:8f:81:c9:7d:79:
         5f:6d:d0:e4:30:a8:9d:e8:0f:fa:7a:fa:f7:6f:34:ac:37:fb:
         21:e6:3f:77:84:ac:a9:03:d7:ce:00:dc:cf:ad:48:4c:e9:18:
         36:92:8d:a4:28:f6:05:9b:86:56:ce:02:14:e5:f0:4f:07:fd:
         95:8d:84:b9:50:31:98:5b:0c:e7:cd:6c:a2:40:81:5a:22:8c:
         84:91:4b:3d:12:80:8c:ad:fc:f3:9f:a1:8c:c1:90:d4:a1:a5:
         86:f4:24:53:9e:ff:dd:87:38:53:0d:7d:6d:51:e0:f5:1a:e7:
         de:fe:ce:9f:af:13:0d:98:60:70:c8:7a:d9:fa:30:51:a2:09:
         a4:89:25:18
-----BEGIN CERTIFICATE-----
MIIDjzCCAnegAwIBAgIIPE3Z6XADrCowDQYJKoZIhvcNAQELBQAwFTETMBEGA1UE
AxMKa3ViZXJuZXRlczAeFw0yMjExMjQxMjQwMDVaFw0yMzExMjQxMjQwMDVaMBkx
FzAVBgNVBAMTDmt1YmUtYXBpc2VydmVyMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEAnvp50DiOLnALqiNH06Ht3HJtUuSt7DQ1AcxiuFBR9QMEJSGjOxMW
PP5B84vbww+dzkxu40YpdLBVtFT4bktDSEvFyqjXA/hmn9kEpVB8N/slK0PruEOx
ImAYbiYQyk9/FV0P1SFOQuNbJVON71I16J4rKCO8XTOQHj3m5tVngVgHr6K44p5/
tVtQWUOU4Q2q96hRzwgfZASplGKO9ZP/cny3ZO9LSs/gl24HtMt5lGYrLjImMJTU
SPWmoz6ULjioaxPmqjMEfYAYJnHtzdFh39CHGeqo6GkN5l7UgDCYIDuDj+TDCfyj
fJJ6GZ5FzRSyz488msZKW20aRQurJdYQ/QIDAQABo4HeMIHbMA4GA1UdDwEB/wQE
AwIFoDATBgNVHSUEDDAKBggrBgEFBQcDATAMBgNVHRMBAf8EAjAAMB8GA1UdIwQY
MBaAFKb3L6IlDKj12H0zNO0xHULUzx6iMIGEBgNVHREEfTB7ggprdWJlcm5ldGVz
ghJrdWJlcm5ldGVzLmRlZmF1bHSCFmt1YmVybmV0ZXMuZGVmYXVsdC5zdmOCJGt1
YmVybmV0ZXMuZGVmYXVsdC5zdmMuY2x1c3Rlci5sb2NhbIIPbWFzdGVyLTIwMjIx
MTI0hwQKYAABhwQKAABHMA0GCSqGSIb3DQEBCwUAA4IBAQAOt2HaQHxOAXIVIx7e
3w8x1wFPBnfVQpl9CvggYP2nd9LsRC3Z8Ww0UZ4KhO9dVOTFRdoGVpYMpw0JDr5R
9kN4p3XKhkcnSE1zJ488sI25XCS0TyVyY44DBC0emTBe25eHC8o/jTPl/Msmo9QY
M8tVX808zbw/udacj4HJfXlfbdDkMKid6A/6evr3bzSsN/sh5j93hKypA9fOANzP
rUhM6Rg2ko2kKPYFm4ZWzgIU5fBPB/2VjYS5UDGYWwznzWyiQIFaIoyEkUs9EoCM
rfzzn6GMwZDUoaWG9CRTnv/dhzhTDX1tUeD1Gufe/s6frxMNmGBwyHrZ+jBRogmk
iSUY
-----END CERTIFICATE-----
```

このcrtファイルによって認証されるのが、二つのIPアドレス（MasterとWorker）と`kubernetes`などのDNS名であることがわかる。

そこで、外部サーバの`/etc/hosts`を編集して、外部サーバが`kubernetes`で名前解決できるようにする。

```/ets/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

143.47.99.xxx kubernetes # 追記
```

Confファイルの`server`もIPアドレスから`kubernetes`に変える。

```conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx==
    server: https://kubernetes:32073 # Change
  name: kubernetes
・・・
```

再度、外部サーバからkubectlコマンドを実行する。

```
external$ kubectl --kubeconfig conf get node
NAME              STATUS   ROLES                  AGE   VERSION
master-20221124   Ready    control-plane,master   7d    v1.23.3
worker-20221124   Ready    <none>                 7d    v1.23.3
```

## Node Restriction
Master ServerでAdmission Pluginのデフォルトを確認する。

```
$ sudo grep admission-plugins /etc/kubernetes/manifests/kube-apiserver.yaml
    - --enable-admission-plugins=NodeRestriction
```

Node Restrictionはデフォルトで有効になっている。

### Admission Controlerの確認
Worker NodeからAdmission Controlerの動作を確認する。

Worker NodeのConfigを確認する。
API Serverにはアクセスできない。
```
$ kubectl config view
apiVersion: v1
clusters: null
contexts: null
current-context: ""
kind: Config
preferences: {}
users: null
$ kubectl get ns
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

kubeletはAPI Serverにアクセスできる。

```
$ sudo cat /etc/kubernetes/kubelet.conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJxxxxxxxxxxxx==
    server: https://10.0.0.71:6443
  name: default-cluster
contexts:
- context:
    cluster: default-cluster
    namespace: default
    user: default-auth
  name: default-context
current-context: default-context
kind: Config
preferences: {}
users:
- name: default-auth
  user:
    client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
    client-key: /var/lib/kubelet/pki/kubelet-client-current.pem
```

kubelet.confをKUBECONFIGに設定する。
※rootユーザで設定する。

```
# export KUBECONFIG=/etc/kubernetes/kubelet.conf
```

```
# kubectl get ns
Error from server (Forbidden): namespaces is forbidden: User "system:node:worker-20221124" cannot list resource "namespaces" in API group "" at the cluster scope
```

API Serverと通信できているが、パーミッションがない。
Worker NodeはCluster Scopeのnamespaceを表示できない。
Nodeは確認できる。

```
# kubectl get node
NAME              STATUS   ROLES                  AGE   VERSION
master-20221124   Ready    control-plane,master   9d    v1.23.3
worker-20221124   Ready    <none>                 9d    v1.23.3
```

Master nodeにLabelを設定するが、権限がない。

```
# kubectl label node master-20221124 test=cks
Error from server (Forbidden): nodes "master-20221124" is forbidden: node "worker-20221124" is not allowed to modify node "master-20221124"
```

Node Restrictionが働いていることがわかる。

Worker Nodeにはラベルが設定できる。

```
# kubectl label node worker-20221124 test=cks
node/worker-20221124 labeled
```
`node-restriction.kubernetes.io`を頭につけると、ラベルが設定できない。

```
# kubectl label node worker-20221124 node-restriction.kubernetes.io/sample=cks
Error from server (Forbidden): nodes "worker-20221124" is forbidden: is not allowed to modify labels: node-restriction.kubernetes.io/sample
```
