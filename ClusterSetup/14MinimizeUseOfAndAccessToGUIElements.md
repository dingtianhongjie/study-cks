# kubernetes Dashboardのインストール
[Github](https://github.com/kubernetes/dashboard)から以下のコマンドを実行

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

確認

```
$ kubectl get ns
NAME                   STATUS   AGE
default                Active   24h
kube-node-lease        Active   24h
kube-public            Active   24h
kube-system            Active   24h
kubernetes-dashboard   Active   107s
$ kubectl -n kubernetes-dashboard get all
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-6f669b9c9b-lps6n   1/1     Running   0          2m34s
pod/kubernetes-dashboard-758765f476-fsgr2        1/1     Running   0          2m34s

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.104.100.165   <none>        8000/TCP   2m34s
service/kubernetes-dashboard        ClusterIP   10.100.221.70    <none>        443/TCP    2m34s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           2m34s
deployment.apps/kubernetes-dashboard        1/1     1            1           2m34s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-6f669b9c9b   1         1         1       2m34s
replicaset.apps/kubernetes-dashboard-758765f476        1         1         1       2m34s
```

# deploymentの編集
[Github](https://github.com/kubernetes/dashboard)から `docs -> common -> dashboard arguments`を参照し、以下のdeploymentを編集する。

```
$ kubectl -n kubernetes-dashboard edit deployment kubernetes-dashboard
deployment.apps/kubernetes-dashboard edited
```

`spec -> containers -> args`の
- `--auto-generate-certificates`を削除
- `--insecure-port=9090` を追記

`spec -> containers -> args -> livenessProbe -> httpGet`の
- `port`を9090に変更
- `scheme`をHTTPに変更

※ここではHTTPでアクセスする。

# Serviceの編集

CludterIPをNodePortに変更する。
```
$ kubectl -n kubernetes-dashboard get svc
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
dashboard-metrics-scraper   ClusterIP   10.104.100.165   <none>        8000/TCP   2d
kubernetes-dashboard        ClusterIP   10.100.221.70    <none>        443/TCP    2d
```

```
$ kubectl -n kubernetes-dashboard edit svc kubernetes-dashboard
service/kubernetes-dashboard edited
$ kubectl -n kubernetes-dashboard get svc
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
dashboard-metrics-scraper   ClusterIP   10.104.100.165   <none>        8000/TCP         2d
kubernetes-dashboard        NodePort    10.100.221.70    <none>        9090:30862/TCP   2d
```
変更内容
```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"k8s-app":"kubernetes-dashboard"},"name":"kubernetes-dashboard","namespace":"kubernetes-dashboard"},"spec":{"ports":[{"port":443,"targetPort":8443}],"selector":{"k8s-app":"kubernetes-dashboard"}}}
  creationTimestamp: "2022-11-18T13:00:31Z"
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
  resourceVersion: "56830"
  uid: 7739a39b-3599-4490-930a-a3aa2d81204b
spec:
  clusterIP: 10.100.221.70
  clusterIPs:
  - 10.100.221.70
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 30862
    port: 9090 #変更
    protocol: TCP
    targetPort: 9090 #変更
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: NodePort #変更
status:
  loadBalancer: {}
```

`http://<worker nodeのIPアドレス>:ポート番号`でダッシュボードにアクセスできることを確認する。
（この時点では権限がないので、画面が見えるだけ）

# RBACの設定

デフォルトのServiceAccountとClusterRoleを確認する。
```
$ kubectl -n kubernetes-dashboard get sa
NAME                   SECRETS   AGE
default                1         2d23h
kubernetes-dashboard   1         2d23h
$ kubectl get clusterrole |grep view
system:aggregate-to-view                                               2022-11-17T12:39:27Z
system:public-info-viewer                                              2022-11-17T12:39:27Z
view                                                                   2022-11-17T12:39:27Z
```

RoleBindingsを作成する。

```
$ kubectl -n kubernetes-dashboard create rolebinding insecure --serviceaccount kubernetes-dashboard:kubernetes-dashboard --clusterrole view
rolebinding.rbac.authorization.k8s.io/insecure created
```

ブラウザをリロードし、「ネームスペース」欄で「kubernetes-dashboard」を指定すると、kubernetes-dashboard namespaceのリソースが確認できる。

同様にClusterRoleBindingsを作成すると、全てのnamespaceのリソースが確認できる。
