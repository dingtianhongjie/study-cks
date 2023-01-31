# マニュアル
https://kubernetes.io/ja/docs/tasks/debug/debug-cluster/audit/

# 学習ログ
## Audit Logの有効化

```
$ sudo mkdir /etc/kubernetes/audit
$ cd /etc/kubernetes/audit/
```
ここに以下のPolicyファイルを作成する。

```
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
```

API Serverのマニフェストを編集する。

```
$ cd ../manifests/
$ sudo vim kube-apiserver.yaml
```

```kube-apiserver.yaml
・・・
spec:
  containers:
  - command:
    - kube-apiserver
    - --audit-policy-file=/etc/kubernetes/audit/policy.yaml       # add
    - --audit-log-path=/etc/kubernetes/audit/logs/audit.log       # add
    - --audit-log-maxsize=500                                     # add
    - --audit-log-maxbackup=5                                     # add
・・・
    volumeMounts:
    - mountPath: /etc/kubernetes/audit      # add
      name: audit                           # add
・・・
  volumes:
  - hostPath:                               # add
      path: /etc/kubernetes/audit           # add
      type: DirectoryOrCreate               # add
    name: audit                             # add
```

API Serverが上がってきたら、確認する。

```
$ ls ../audit/
logs  policy.yaml
```

Logファイルが作成されて、ログが記録されている。

```
# cd /etc/kubernetes/audit/logs/
# ls -l
total 3156
-rw------- 1 root root 3229063 Jan 29 13:23 audit.log
# tail -1 audit.log
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"8c78a368-f65c-4a78-ac41-0b26b7042ade","stage":"ResponseComplete","requestURI":"/readyz","verb":"get","user":{"username":"system:anonymous","groups":["system:unauthenticated"]},"sourceIPs":["10.0.0.186"],"userAgent":"kube-probe/1.25","responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2023-01-29T13:23:42.912872Z","stageTimestamp":"2023-01-29T13:23:42.914467Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by ClusterRoleBinding \"system:public-info-viewer\" of ClusterRole \"system:public-info-viewer\" to Group \"system:unauthenticated\""}}
```

## リソース（Secret）を作った時のLogを確認する。

Audit logを確認しておく。
```
# tail -f audit.log | grep secure
```

別ターミナルでSecretを作成する。

```
$ kubectl create secret generic secure --from-literal user=admin
secret/secure created
```

Audit logにかかれる。

→JSONだけど一行で吐かれる。

```
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"c30f989b-a894-4e75-8aa9-3f42b5478e5d","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/default/secrets?fieldManager=kubectl-create\u0026fieldValidation=Strict","verb":"create","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["10.0.0.186"],"userAgent":"kubectl/v1.25.4 (linux/amd64) kubernetes/872a965","objectRef":{"resource":"secrets","namespace":"default","name":"secure","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestReceivedTimestamp":"2023-01-30T12:36:48.209351Z","stageTimestamp":"2023-01-30T12:36:48.213395Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
```

成形すると、こうなる。

```
{
	"kind": "Event",
	"apiVersion": "audit.k8s.io/v1",
	"level": "Metadata",
	"auditID": "c30f989b-a894-4e75-8aa9-3f42b5478e5d",
	"stage": "ResponseComplete",
	"requestURI": "/api/v1/namespaces/default/secrets?fieldManager=kubectl-create&fieldValidation=Strict",
	"verb": "create",
	"user": {
		"username": "kubernetes-admin",
		"groups": [
			"system:masters",
			"system:authenticated"
		]
	},
	"sourceIPs": [
		"10.0.0.186"
	],
	"userAgent": "kubectl/v1.25.4 (linux/amd64) kubernetes/872a965",
	"objectRef": {
		"resource": "secrets",
		"namespace": "default",
		"name": "secure",
		"apiVersion": "v1"
	},
	"responseStatus": {
		"metadata": {},
		"code": 201
	},
	"requestReceivedTimestamp": "2023-01-30T12:36:48.209351Z",
	"stageTimestamp": "2023-01-30T12:36:48.213395Z",
	"annotations": {
		"authorization.k8s.io/decision": "allow",
		"authorization.k8s.io/reason": ""
	}
}
```

 ## Audit Policyのカスタマイズ
 Policyファイルを以下のように編集する。

 ```policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy

# 全てのリクエストのRequestReceived (監査ハンドラーがリクエストを受信すると同時に生成されるイベントのステージ)でログを生成しない。
omitStages:
  - "RequestReceived"
rules:

# "get", "list", "watch"の監査ログを取得しない。
  - level: None
    verbs: ["get", "list", "watch"]

# Secretのリクエストのメタデータを記録する
  - level: Metadata
    resources:
    - group: ""
      resources: ["secrets"]

# その他全てはRequestResponse（イベントのメタデータ、リクエストとレスポンスのボディを記録） レベルで記録する。
  - level: RequestResponse
 ```

 API Serverのマニフェストを編集して、APIサーバを再起動させる。

 audit.logを確認し、想定通りに監査ログが吐かれていることを確認する。

