# マニュアル
https://kubernetes.io/docs/concepts/services-networking/ingress/

https://kubernetes.io/ja/docs/concepts/services-networking/ingress-controllers/

https://kubernetes.github.io/ingress-nginx/deploy/

# 学習ログ
## Ingress ControllerとIngressのデプロイ

Install Guideからマニフェストをダウンロード
```
$ wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml
--2023-02-03 07:48:19--  https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.108.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15760 (15K) [text/plain]
Saving to: ‘deploy.yaml’

deploy.yaml                                                 100%[========================================================================================================================================>]  15.39K  --.-KB/s    in 0s

2023-02-03 07:48:19 (79.2 MB/s) - ‘deploy.yaml’ saved [15760/15760]
```

Install GuideのマニフェストはLoadBalancerになっているので、環境に合わせてNodePortに変える。

```
・・・
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.5.1
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  externalTrafficPolicy: Local
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - appProtocol: http
    name: http
    port: 80
    protocol: TCP
    targetPort: http
  - appProtocol: https
    name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: NodePort # LoadBalancerから変更
・・・
```

apply する。

```
$ kubectl apply -f deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
```
```
$ kubectl -n ingress-nginx get all
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-9xrbz        0/1     Completed   0          7m30s
pod/ingress-nginx-admission-patch-szfnl         0/1     Completed   1          7m30s
pod/ingress-nginx-controller-8574b6d7c9-ds9h9   1/1     Running     0          7m30s

NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             NodePort    10.99.139.180   <none>        80:31647/TCP,443:31966/TCP   7m30s
service/ingress-nginx-controller-admission   ClusterIP   10.97.32.43     <none>        443/TCP                      7m30s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           7m30s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-8574b6d7c9   1         1         1       7m30s

NAME                                       COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   1/1           5s         7m30s
job.batch/ingress-nginx-admission-patch    1/1           6s         7m30s
```

外部端末からWorkerノードのIPアドレスとNodePortのPortを指定して、疎通を確認する。

```
$ curl http://150.230.169.224:31647
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

IngressClassを確認する。

```
$ kubectl get ingressclasses.networking.k8s.io
NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       4m31s
```

デフォルトのIngressClassを指定して、Ingressをデプロイする。

```ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /service1
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80

      - path: /service2
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 80
```

失敗する

```
$ kubectl apply -f ingress.yaml
Error from server (InternalError): error when creating "ingress.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": failed to call webhook: Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses?timeout=10s": context deadline exceeded
```

Webhookがタイムアウトしてしまうので、このValidatingWebhookConfigurationを削除する。

（この対処が正しいのかは不明）

```
$ kubectl get ValidatingWebhookConfiguration
NAME                      WEBHOOKS   AGE
ingress-nginx-admission   1          7m24s
$ kubectl get ValidatingWebhookConfiguration -o yaml
apiVersion: v1
items:
- apiVersion: admissionregistration.k8s.io/v1
  kind: ValidatingWebhookConfiguration
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"admissionregistration.k8s.io/v1","kind":"ValidatingWebhookConfiguration","metadata":{"annotations":{},"labels":{"app.kubernetes.io/component":"admission-webhook","app.kubernetes.io/instance":"ingress-nginx","app.kubernetes.io/name":"ingress-nginx","app.kubernetes.io/part-of":"ingress-nginx","app.kubernetes.io/version":"1.5.1"},"name":"ingress-nginx-admission"},"webhooks":[{"admissionReviewVersions":["v1"],"clientConfig":{"service":{"name":"ingress-nginx-controller-admission","namespace":"ingress-nginx","path":"/networking/v1/ingresses"}},"failurePolicy":"Fail","matchPolicy":"Equivalent","name":"validate.nginx.ingress.kubernetes.io","rules":[{"apiGroups":["networking.k8s.io"],"apiVersions":["v1"],"operations":["CREATE","UPDATE"],"resources":["ingresses"]}],"sideEffects":"None"}]}
    creationTimestamp: "2023-02-03T07:50:41Z"
    generation: 2
    labels:
      app.kubernetes.io/component: admission-webhook
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
      app.kubernetes.io/version: 1.5.1
    name: ingress-nginx-admission
    resourceVersion: "212250"
    uid: 849e9ae5-ac55-40f9-bb12-fbbe12538d95
  webhooks:
  - admissionReviewVersions:
    - v1
    clientConfig:
      caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkVENDQVJ5Z0F3SUJBZ0lSQUxET1JBNWFGYStQalhsVGlQMjc1Wjh3Q2dZSUtvWkl6ajBFQXdJd0R6RU4KTUFzR0ExVUVDaE1FYm1sc01UQWdGdzB5TXpBeU1ETXdOelExTkRKYUdBOHlNVEl6TURFeE1EQTNORFUwTWxvdwpEekVOTUFzR0ExVUVDaE1FYm1sc01UQlpNQk1HQnlxR1NNNDlBZ0VHQ0NxR1NNNDlBd0VIQTBJQUJMU3EzRUNKCmptbGh3dVZqL3MxS2EvWTUwRkg5ZDV3UCtSaVVSc2EwUjFyZFBKM1N1c3ErblVpVmxicmdNZEtUcVlXMDRpeHkKT0dEa3dCR01RZVptb0lxalZ6QlZNQTRHQTFVZER3RUIvd1FFQXdJQ0JEQVRCZ05WSFNVRUREQUtCZ2dyQmdFRgpCUWNEQVRBUEJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJRQXAyMHprc1Bvd0F3WXdVaWUxM3I4CmtDSGJoREFLQmdncWhrak9QUVFEQWdOSEFEQkVBaUF6WE9aSjRTMHZlWEs1bnVUbnZield3b3laYzlVR3VJZzcKZFBhSGhaQzBIQUlnSTBlckJJL3NvSkpyMWtQMnhCQnFiZFRDTXJuaG1aT0RtMXpLZWJWUGZJMD0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
      service:
        name: ingress-nginx-controller-admission
        namespace: ingress-nginx
        path: /networking/v1/ingresses
        port: 443
    failurePolicy: Fail
    matchPolicy: Equivalent
    name: validate.nginx.ingress.kubernetes.io
    namespaceSelector: {}
    objectSelector: {}
    rules:
    - apiGroups:
      - networking.k8s.io
      apiVersions:
      - v1
      operations:
      - CREATE
      - UPDATE
      resources:
      - ingresses
      scope: '*'
    sideEffects: None
    timeoutSeconds: 10
kind: List
metadata:
  resourceVersion: ""
```

```
$ kubectl delete ValidatingWebhookConfiguration ingress-nginx-admission
validatingwebhookconfiguration.admissionregistration.k8s.io "ingress-nginx-admission" deleted
```

再度Ingressをデプロイする。

```
$ kubectl apply -f ingress.yaml
ingress.networking.k8s.io/secure-ingress created
$ kubectl get ingress
NAME             CLASS   HOSTS   ADDRESS         PORTS   AGE
secure-ingress   nginx   *       10.99.139.180   80      4m22s
```

動作確認用に2つのPodとClusterIPをデプロイする。

```
$ kubectl run pod1 --image nginx
pod/pod1 created
$ kubectl run pod2 --image httpd
pod/pod2 created
$ kubectl expose pod pod1 --port 80 --name service1
service/service1 exposed
$ kubectl expose pod pod2 --port 80 --name service2
service/service2 exposed
```
```
$ kubectl get pod,svc
NAME       READY   STATUS    RESTARTS   AGE
pod/pod1   1/1     Running   0          51s
pod/pod2   1/1     Running   0          42s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   13m
service/service1     ClusterIP   10.107.254.108   <none>        80/TCP    18s
service/service2     ClusterIP   10.100.35.135    <none>        80/TCP    10s
```

外部端末から疎通を確認する。

```
$ curl http://150.230.169.224:31647/service1
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
```

```
$ curl http://150.230.169.224:31647/service2
<html><body><h1>It works!</h1></body></html>
```

## httpsでのアクセス

NodePortのhttps（443）のポートを確認
```
$ kubectl -n ingress-nginx get svc
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.99.139.180   <none>        80:31647/TCP,443:31966/TCP   16m
ingress-nginx-controller-admission   ClusterIP   10.97.32.43     <none>        443/TCP                      16m
```

httpsでアクセスする

```
$ curl https://150.230.169.224:31966/service2
curl: (60) SSL certificate problem: self signed certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
$ curl https://150.230.169.224:31966/service2 -k
<html><body><h1>It works!</h1></body></html>
```

-kで証明書を受け入れるとアクセスできる。

-kvで詳細を確認する。

```
$ curl https://150.230.169.224:31966/service2 -kv
*   Trying 150.230.169.224:31966...
* TCP_NODELAY set
* Connected to 150.230.169.224 (150.230.169.224) port 31966 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
*  start date: Feb  3 12:37:45 2023 GMT
*  expire date: Feb  3 12:37:45 2024 GMT
*  issuer: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x55bbb6a0d8c0)
> GET /service2 HTTP/2
> Host: 150.230.169.224:31966
> user-agent: curl/7.68.0
> accept: */*
>
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
< HTTP/2 200
< date: Fri, 03 Feb 2023 13:00:24 GMT
< content-type: text/html
< content-length: 45
< last-modified: Mon, 11 Jun 2007 18:53:14 GMT
< etag: "2d-432a5e4a73a80"
< accept-ranges: bytes
< strict-transport-security: max-age=15724800; includeSubDomains
<
<html><body><h1>It works!</h1></body></html>
* Connection #0 to host 150.230.169.224 left intact
```

`Fake Certificate`が設定されている。

自己証明書とキーを作成する。

```
$ openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
Generating a RSA private key
......................................................................................................................................................................................................++++
..........................................................................................................++++
writing new private key to 'key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:secure-ingress.com
Email Address []:
```

```
$ ls -l
total 28
-rw-rw-r-- 1 ubuntu ubuntu  2017 Feb  3 13:04 cert.pem
-rw-rw-r-- 1 ubuntu ubuntu 15756 Feb  3 12:37 deploy.yaml
-rw-rw-r-- 1 ubuntu ubuntu   520 Feb  3 12:38 ingress.yaml
-rw------- 1 ubuntu ubuntu  3272 Feb  3 13:03 key.pem
```

Secretを作成する。

```
$ kubectl create secret tls secure-ingress --cert=cert.pem --key=key.pem
secret/secure-ingress created
```

このSecretを指定して、Ingressを再作成する。

```ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - secure-ingress.com # opensslのCommonNameで指定したホスト名
    secretName: secure-ingress
  rules:
  - host: secure-ingress.com # opensslのCommonNameで指定したホスト名
    http:
      paths:
      - path: /service1
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80

      - path: /service2
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 80
```

```
$ kubectl apply -f ingress.yaml
ingress.networking.k8s.io/secure-ingress configured
$ kubectl get ingress
NAME             CLASS   HOSTS                ADDRESS         PORTS     AGE
secure-ingress   nginx   secure-ingress.com   10.99.139.180   80, 443   31m
```

ホスト名を指定して確認する。

```
$ curl https://secure-ingress.com:31966/service2 -kv --resolve secure-ingress.com:31966:150.230.169.224
* Added secure-ingress.com:31966:150.230.169.224 to DNS cache
* Hostname secure-ingress.com was found in DNS cache
*   Trying 150.230.169.224:31966...
* TCP_NODELAY set
* Connected to secure-ingress.com (150.230.169.224) port 31966 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=AU; ST=Some-State; O=Internet Widgits Pty Ltd; CN=secure-ingress.com
*  start date: Feb  3 13:04:23 2023 GMT
*  expire date: Feb  3 13:04:23 2024 GMT
*  issuer: C=AU; ST=Some-State; O=Internet Widgits Pty Ltd; CN=secure-ingress.com
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x56340a6128c0)
> GET /service2 HTTP/2
> Host: secure-ingress.com:31966
> user-agent: curl/7.68.0
> accept: */*
>
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
< HTTP/2 200
< date: Fri, 03 Feb 2023 13:18:40 GMT
< content-type: text/html
< content-length: 45
< last-modified: Mon, 11 Jun 2007 18:53:14 GMT
< etag: "2d-432a5e4a73a80"
< accept-ranges: bytes
< strict-transport-security: max-age=15724800; includeSubDomains
<
<html><body><h1>It works!</h1></body></html>
* Connection #0 to host secure-ingress.com left intact
```

secure-ingress.comでcertificateされている。