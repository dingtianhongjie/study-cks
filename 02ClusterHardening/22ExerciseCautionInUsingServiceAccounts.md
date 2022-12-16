# マニュアル

https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/
https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account


# 学習ログ
## Pod内でのserviceaccountの利用
serviceaccountを作成する。
```
$ kubectl create sa sa01
serviceaccount/sa01 created
$ kubectl get sa
NAME      SECRETS   AGE
default   0         23h
sa01      0         4s
```

作成したserviceaccountを使ってAPI Serverに認証するためにトークンを作成する。

```
$ kubectl create token sa01
eyJhbGciOiJSUzI1NiIsImtpZCI6IllMZldRMzBvVXR2YV9ZNXdaX3NWWTEwN2RlVmNKZlVPSE1VWmtOTF9aWTQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjcxMTEyNTI1LCJpYXQiOjE2NzExMDg5MjUsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6InNhMDEiLCJ1aWQiOiJjZTRmOWYxOC1iNGY0LTQxZGItOGJkNi1lMTk2ZGQzMTM0MTgifX0sIm5iZiI6MTY3MTEwODkyNSwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6c2EwMSJ9.N_1FPtf0L6u7ADL0EmgHhsHOsPInE50HZGqmj8l8ZsFfjqYAnQoy2oKHX9UTh3HF1ImzFmIOHeIpkZ-ABDF6wY8yUJlK4FoX4kWK_eavRTgRC-zMfmlO8Io7IXnCAcmNPZjWbm4sSR6l6NTVARRFWXQCyIuHTSSqq1PgnL8FAD_Yp2L2Jh4HRf4Otj-U-n3HiFnKIbLRl5pD-LzQqSXRGLDvXOJWhf3_XjBJOnVGbBkU7nXKF4MIiB6PCrgG81wKk4lBhdPWi7tP04pxpMOT8VZa9eWcoCrhmxVrgr4e_WfdWqiR7uhG9nSzKvvCJHNNo3Yq3Pdv5g_l_GYhNTIGqQ
```

ちなみに、コマンド実行するたびに、異なるトークンが作成される。

作成したserviceaccountを指定してPodを作成する。

```pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  serviceAccountName: sa01
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
$ kubectl apply -f pod.yaml
pod/nginx created
$ kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          93s
```

Podにログインしてマウントしているserviceaccountを確認する。

```
$ kubectl exec -it nginx -- bash
root@nginx:/# mount |grep service
tmpfs on /run/secrets/kubernetes.io/serviceaccount type tmpfs (ro,relatime,size=16278440k,inode64)
root@nginx:/# cd /run/secrets/kubernetes.io/serviceaccount
root@nginx:/run/secrets/kubernetes.io/serviceaccount# ls
ca.crt  namespace  token
```

tokenを確認する。

```
root@nginx:/run/secrets/kubernetes.io/serviceaccount# cat token
eyJhbGciOiJSUzI1NiIsImtpZCI6IllMZldRMzBvVXR2YV9ZNXdaX3NWWTEwN2RlVmNKZlVPSE1VWmtOTF9aWTQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzAyNjQ1NDY2LCJpYXQiOjE2NzExMDk0NjYsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0IiwicG9kIjp7Im5hbWUiOiJuZ2lueCIsInVpZCI6Ijk4NDU1MmQ2LWI3YjktNDdmOC05ZTkwLTgxMmMyNDQ2YTllZSJ9LCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoic2EwMSIsInVpZCI6ImNlNGY5ZjE4LWI0ZjQtNDFkYi04YmQ2LWUxOTZkZDMxMzQxOCJ9LCJ3YXJuYWZ0ZXIiOjE2NzExMTMwNzN9LCJuYmYiOjE2NzExMDk0NjYsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OnNhMDEifQ.NT98FGmWZ67EyR05DCTOMqfTNPLVn7jq1RFAJ-soiO11vRJZ8s-WNSatrmHXhKMEQcloOprU7YKZoEYnB3o6I3vqOTQKArE90QC94KVaQDh4ua9pNhKRMYGOgNpWlX8SUwMLrHIl8GNV9W03-jwVKJF1HqYaGtcAC1hB-wheu2_mqbmBhUcftHARGODk35P7btsLF-3dkizOwdbIYRgVrAuxBGfie_LCzAo6iVRX0y5B0bsSuAF9brUehycyM0SaRutnoBh-ib43-l3QAhNwfTmCz53tCdErRKe0XYgzMCUOUCF8HBYlj8Jl7wmDoKV6AoNd2ALFhbecnnTLYY1YoA
```

`kubectl token`で作成したトークンと異なるが、以下のサイトで調べると、同じserviceaccountで作成されたものだとわかる。

https://jwt.io/

Pod内部からserviceaccountを使用してAPI Serverにアクセスする。
API Server（SERVICE_HOST）のIPアドレスを確認して、curlでアクセスする。

```
root@nginx:~# env |grep KUBER
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
root@nginx:~# curl https://10.96.0.1
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

証明書の問題で失敗する。

```
root@nginx:~# curl https://10.96.0.1 -k
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

-kで無視すると、Forbiddenになる。

-Hでトークンを指定してアクセスする。

```
root@nginx:~# curl https://10.96.0.1 -k -H "Authorization: Bearer $(cat /run/secrets/kubernetes.io/serviceaccount/token)"
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "forbidden: User \"system:serviceaccount:default:sa01\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {},
  "code": 403
}
```

同じForbiddenだが、serviceaccountで認証できている。
RBACでこのserviceaccountにroleを割り当ててると、Podの中からKubernetesリソースのgetやdeleteができるようになる。

## serviceaccountのマウントの無効化

先ほど作成したPodのマニュフェストを以下のように書き換えて、Podを作り直す。

```pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  serviceAccountName: sa01
  automountServiceAccountToken: false # added
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
$ kubectl replace -f pod.yaml --force
pod "nginx" deleted
pod/nginx replaced
```

Podにログインして、sericeaccountをマウントしてないことを確認する。

```
$ kubectl exec -it nginx -- bash
root@nginx:/# mount |grep service
root@nginx:/#
```