# マニュアル
https://kubernetes.io/ja/docs/concepts/services-networking/network-policies/

# 学習ログ
## 事前準備

Podを2つデプロイする。

```
$ kubectl run frontend --image=nginx
pod/frontend created
$ kubectl run backend --image=nginx
pod/backend created
```

各PodにServiceを作成する。

```
$ kubectl expose pod frontend --port 80
service/frontend exposed
ubuntu@master-20221117:~/06$ kubectl expose pod backend --port 80
service/backend exposed
```

確認

```
$ kubectl get all -o wide
NAME           READY   STATUS    RESTARTS   AGE     IP              NODE              NOMINATED NODE   READINESS GATES
pod/backend    1/1     Running   0          4m7s    192.168.11.67   worker-20221117   <none>           <none>
pod/frontend   1/1     Running   0          4m15s   192.168.11.66   worker-20221117   <none>           <none>

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE     SELECTOR
service/backend      ClusterIP   10.100.32.199   <none>        80/TCP    2m25s   run=backend
service/frontend     ClusterIP   10.111.250.46   <none>        80/TCP    2m34s   run=frontend
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   32m     <none>
```

疎通を確認

- frontend -> backend
```
$ kubectl exec frontend -- curl 192.168.11.67
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
100   615  100   615    0     0   600k      0 --:--:-- --:--:-- --:--:--  600k

```

- backend -> frontend
```
$ kubectl exec backend -- curl 192.168.11.66
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   615  100   615    0     0   600k      0 --:--:-- --:--:-- --:--:--  600k
```

## デフォルトのNetwork policy

[全てのトラフィックを拒否する。](https://kubernetes.io/ja/docs/concepts/services-networking/network-policies/#%E3%83%87%E3%83%95%E3%82%A9%E3%83%AB%E3%83%88%E3%81%A7%E5%86%85%E5%90%91%E3%81%8D%E3%81%A8%E5%A4%96%E5%90%91%E3%81%8D%E3%81%AE%E3%81%99%E3%81%B9%E3%81%A6%E3%81%AE%E3%83%88%E3%83%A9%E3%83%95%E3%82%A3%E3%83%83%E3%82%AF%E3%82%92%E6%8B%92%E5%90%A6%E3%81%99%E3%82%8B)

```default-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

```
$ kubectl apply -f default-deny.yaml
networkpolicy.networking.k8s.io/default-deny-all created
$ kubectl get networkpolicies
NAME               POD-SELECTOR   AGE
default-deny-all   <none>         35s
```

疎通できないことを確認
- frontend -> backend
```
$ kubectl exec frontend -- curl 192.168.11.67
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:09 --:--:--     0^C
```

- backend -> frontend
```
$ kubectl exec backend -- curl 192.168.11.66
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:06 --:--:--     0^C
``` 

## Network Policyによる制御
### PodSelector

- frontendからBackendへのEgressを許可する。

```frontend.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          run: backend
```

```
$ kubectl apply -f frontend.yaml
networkpolicy.networking.k8s.io/frontend created
```

- frontendからBackendへのIngressを許可する。
```backend.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: frontend
```

```
$ kubectl apply -f backend.yaml
networkpolicy.networking.k8s.io/backend created
$ kubectl get networkpolicies
NAME               POD-SELECTOR   AGE
backend            run=backend    70s
default-deny-all   <none>         15m
frontend           run=frontend   6m1s
```

frontendからbackendへ疎通できることを確認する。
```
$ kubectl exec frontend -- curl 192.168.11.67
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   615  100   615    0     0   600k      0 --:--:-- --:--:-- --:--:--  600k
```

backendからfrontendへ疎通できないことを確認する。
```
$ kubectl exec backend -- curl 192.168.11.66
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:09 --:--:--     0^C
```

### namespaceSelector
namespaceを作成する。
```
$ kubectl create ns db
namespace/db created
```

作成したnamespaceにlabelを設定する。

```
$ kubectl edit ns db
namespace/db edited
```
以下のようにlabelsを追加する。

```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2022-11-18T06:52:25Z"
  labels:
    kubernetes.io/metadata.name: db
  name: db
  resourceVersion: "14653"
  uid: 2be612ec-905a-4f2d-ac95-5c00ead6e54f
  labels:　#追記
    ns: db #追記
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
```

確認
```
$ kubectl get ns -L ns
NAME              STATUS   AGE     NS
db                Active   4m35s   db
default           Active   18h
kube-node-lease   Active   18h
kube-public       Active   18h
kube-system       Active   18h
```

Podを作成する。

```
$ kubectl -n db run db --image=nginx
pod/db created
$ kubectl -n db get pod -o wide
NAME   READY   STATUS    RESTARTS   AGE     IP              NODE              NOMINATED NODE   READINESS GATES
db     1/1     Running   0          7m37s   192.168.11.71   worker-20221117   <none>           <none>
```

backendからdbへ疎通できないことを確認する。
（この時点では、backendからのEgressを許可していない。）

```
$ kubectl exec backend -- curl 192.168.11.71
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:04 --:--:--     0^C
```

backend network policyを以下のように編集する。
namespaceSelectorでEgressを許可する。

```backend.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: frontend
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          ns: db
```


```
$ kubectl apply -f backend.yaml
networkpolicy.networking.k8s.io/backend configured
```

backendからdbへ疎通できることを確認する。
```
$ kubectl exec backend -- curl 192.168.11.71
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
100   615  100   615    0     0   300k      0 --:--:-- --:--:-- --:--:--  600k
```

db namespaceのdefault denyを設定

```db-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-deny-all
  namespace: db
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```
```
$ kubectl apply -f db-deny.yaml
networkpolicy.networking.k8s.io/db-deny-all created
$ kubectl get networkpolicies -A
NAMESPACE   NAME               POD-SELECTOR   AGE
db          db-deny-all        <none>         103s
default     backend            run=backend    17h
default     default-deny-all   <none>         18h
default     frontend           run=frontend   17h
```

backendからdbへ疎通できなくなったことを確認

```
$ kubectl exec backend -- curl 192.168.11.71
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:05 --:--:--     0^C
```

default namespaceからのIngressを許可する。

```db.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db
  namespace: db
spec:
  podSelector:
    matchLabels:
      run: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          ns: default
```

```
$ kubectl apply -f db.yaml
networkpolicy.networking.k8s.io/db created
```

default namespaceにlabelを設定する。

```
$ kubectl edit ns default
namespace/default edited
```

```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2022-11-17T12:39:27Z"
  labels:
    kubernetes.io/metadata.name: default
  name: default
  resourceVersion: "192"
  uid: 79e99b7f-dd10-4e87-ab8e-a9e558a05cd0
  labels: #追記
    ns: default #追記
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
```

backendからdbへ疎通できることを確認する

```
$ kubectl exec backend -- curl 192.168.11.71
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
100   615  100   615    0     0   600k      0 --:--:-- --:--:-- --:--:--  600k
```
