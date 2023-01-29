# マニュアル
https://kubernetes.io/ja/docs/tasks/debug/debug-cluster/audit/

# 学習ログ

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