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