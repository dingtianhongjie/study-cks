# マニュアル

https://docs.oracle.com/ja-jp/iaas/Content/Compute/Tasks/gettingmetadata.htm

# 学習ログ
## メタデータの確認

Kubernetesクラスタを構成するインスタンスのメタデータを取得する。
今回はOracle Cloudを使用しているので、Oracleのマニュアルで取得方法を確認する。

master nodeからメタデータを取得できる。

```
$ curl -H "Authorization: Bearer Oracle" -L http://169.254.169.254/opc/v2/instance/
{
  "agentConfig": {
    "allPluginsDisabled": false,
    "managementDisabled": false,
    "monitoringDisabled": false,
    "pluginsConfig": [
      {
        "desiredState": "DISABLED",
        "name": "Vulnerability Scanning"
・・・
```

テスト用のPodをデプロイする。

```
$ kubectl run nginx --image nginx
pod/nginx created
$ kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          11s
```

Podにログインして、同様にメタデータを取得できることを確認する。

```
$ kubectl exec nginx -it -- bash
root@nginx:/# curl -H "Authorization: Bearer Oracle" -L http://169.254.169.254/opc/v2/instance/
{
  "agentConfig": {
    "allPluginsDisabled": false,
    "managementDisabled": false,
    "monitoringDisabled": false,
    "pluginsConfig": [
      {
        "desiredState": "DISABLED",
        "name": "Vulnerability Scanning"
・・・
```

## Podからmetadataへのアクセスを拒否する。

全てのPodのメタデータ(169.254.169.254)へのアクセスを拒否する。

```deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cloud-metadata-deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32
```

```
$ kubectl apply -f deny.yaml
networkpolicy.networking.k8s.io/cloud-metadata-deny created
```

Podにログインし、アクセスできないことを確認する。

```
$ kubectl exec nginx -it -- bash
root@nginx:/# curl -H "Authorization: Bearer Oracle" -L http://169.254.169.254/opc/v2/instance/
^C
```

master nodeからは引き続きアクセスできる。
```
$ curl -H "Authorization: Bearer Oracle" -L http://169.254.169.254/opc/v2/instance/
{
  "agentConfig": {
    "allPluginsDisabled": false,
    "managementDisabled": false,
    "monitoringDisabled": false,
    "pluginsConfig": [
      {
        "desiredState": "DISABLED",
        "name": "Vulnerability Scanning"
```

## 特定のLabelを持つPodはmetadataへのアクセスを許可する。

```allow.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cloud-metadata-allow
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: nginx
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 169.254.169.254/32
```
```
$ kubectl apply -f allow.yaml
networkpolicy.networking.k8s.io/cloud-metadata-allow created
```

Podにログインして、メタデータを取得できることを確認する。

```
$ kubectl exec nginx -it -- bash
root@nginx:/# curl -H "Authorization: Bearer Oracle" -L http://169.254.169.254/opc/v2/instance/
{
  "agentConfig": {
    "allPluginsDisabled": false,
    "managementDisabled": false,
    "monitoringDisabled": false,
    "pluginsConfig": [
      {
        "desiredState": "DISABLED",
        "name": "Vulnerability Scanning"
```

Podのラベルを変更して取得できないことを確認する。

```
$ kubectl edit pod nginx
pod/nginx edited
$ kubectl get pod --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
nginx   1/1     Running   0          27m   run=nginxnginx
$ kubectl exec nginx -it -- bash
root@nginx:/# curl -H "Authorization: Bearer Oracle" -L http://169.254.169.254/opc/v2/instance/
^C
```
