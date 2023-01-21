# マニュアル
https://kubesec.io/

# 学習ログ

## kubesec
Podのマニフェストファイルを作って、Kubesecでスキャンする。
```
$ kubectl run nginx --image nginx -o yaml --dry-run=client >pod.yaml
```
```
$ sudo docker run -i kubesec/kubesec:512c5e0 scan /dev/stdin < pod.yaml
Unable to find image 'kubesec/kubesec:512c5e0' locally
512c5e0: Pulling from kubesec/kubesec
c87736221ed0: Pull complete
5dfbfe40753f: Pull complete
0ab7f5410346: Pull complete
b91424b4f19c: Pull complete
0cff159cca1a: Pull complete
32836ab12770: Pull complete
Digest: sha256:8b1e0856fc64cabb1cf91fea6609748d3b3ef204a42e98d0e20ebadb9131bcb7
Status: Downloaded newer image for kubesec/kubesec:512c5e0
[
  {
    "object": "Pod/nginx.default",
    "valid": true,
    "message": "Passed with a score of 0 points",
    "score": 0,
    "scoring": {
      "advise": [
        {
          "selector": "containers[] .securityContext .capabilities .drop",
          "reason": "Reducing kernel capabilities available to a container limits its attack surface"
        },
        {
          "selector": "containers[] .securityContext .capabilities .drop | index(\"ALL\")",
          "reason": "Drop all capabilities and add only those required to reduce syscall attack surface"
        },
        {
          "selector": "containers[] .securityContext .readOnlyRootFilesystem == true",
          "reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost"
        },
        {
          "selector": "containers[] .securityContext .runAsNonRoot == true",
          "reason": "Force the running image to run as a non-root user to ensure least privilege"
        },
        {
          "selector": "containers[] .resources .requests .cpu",
          "reason": "Enforcing CPU requests aids a fair balancing of resources across the cluster"
        },
        {
          "selector": "containers[] .securityContext .runAsUser -gt 10000",
          "reason": "Run as a high-UID user to avoid conflicts with the host's user table"
        },
        {
          "selector": "containers[] .resources .limits .cpu",
          "reason": "Enforcing CPU limits prevents DOS via resource exhaustion"
        },
        {
          "selector": ".metadata .annotations .\"container.seccomp.security.alpha.kubernetes.io/pod\"",
          "reason": "Seccomp profiles set minimum privilege and secure against unknown threats"
        },
        {
          "selector": "containers[] .resources .requests .memory",
          "reason": "Enforcing memory requests aids a fair balancing of resources across the cluster"
        },
        {
          "selector": "containers[] .resources .limits .memory",
          "reason": "Enforcing memory limits prevents DOS via resource exhaustion"
        },
        {
          "selector": ".metadata .annotations .\"container.apparmor.security.beta.kubernetes.io/nginx\"",
          "reason": "Well defined AppArmor policies may provide greater protection from unknown threats. WARNING: NOT PRODUCTION READY"
        },
        {
          "selector": ".spec .serviceAccountName",
          "reason": "Service accounts restrict Kubernetes API access and should be configured with least privilege"
        }
      ]
    }
  }
]
```

パスはしたけど、スコアが０なので修正する。
SecurityContextでrunAsNonRootをTrueにする。

```pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
    securityContext:
      runAsNonRoot: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

再度実行すると、スコアが"1"になっている。

```
$ sudo docker run -i kubesec/kubesec:512c5e0 scan /dev/stdin < pod.yaml
[
  {
    "object": "Pod/nginx.default",
    "valid": true,
    "message": "Passed with a score of 1 points",
    "score": 1,
    "scoring": {
      "advise": [
        {
          "selector": "containers[] .securityContext .readOnlyRootFilesystem == true",
          "reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost"
        },
        {
          "selector": "containers[] .securityContext .runAsUser -gt 10000",
          "reason": "Run as a high-UID user to avoid conflicts with the host's user table"
        },
        {
          "selector": "containers[] .securityContext .capabilities .drop",
          "reason": "Reducing kernel capabilities available to a container limits its attack surface"
        },
        {
          "selector": "containers[] .resources .limits .memory",
          "reason": "Enforcing memory limits prevents DOS via resource exhaustion"
        },
        {
          "selector": ".spec .serviceAccountName",
          "reason": "Service accounts restrict Kubernetes API access and should be configured with least privilege"
        },
        {
          "selector": "containers[] .securityContext .capabilities .drop | index(\"ALL\")",
          "reason": "Drop all capabilities and add only those required to reduce syscall attack surface"
        },
        {
          "selector": ".metadata .annotations .\"container.seccomp.security.alpha.kubernetes.io/pod\"",
          "reason": "Seccomp profiles set minimum privilege and secure against unknown threats"
        },
        {
          "selector": ".metadata .annotations .\"container.apparmor.security.beta.kubernetes.io/nginx\"",
          "reason": "Well defined AppArmor policies may provide greater protection from unknown threats. WARNING: NOT PRODUCTION READY"
        },
        {
          "selector": "containers[] .resources .requests .cpu",
          "reason": "Enforcing CPU requests aids a fair balancing of resources across the cluster"
        },
        {
          "selector": "containers[] .resources .limits .cpu",
          "reason": "Enforcing CPU limits prevents DOS via resource exhaustion"
        },
        {
          "selector": "containers[] .resources .requests .memory",
          "reason": "Enforcing memory requests aids a fair balancing of resources across the cluster"
        }
      ]
    }
  }
]
```

## OPAによるマニフェストファイルのチェック

サンプルのダウンロード
```
$ git clone https://github.com/killer-sh/cks-course-environment.git
Cloning into 'cks-course-environment'...
remote: Enumerating objects: 467, done.
remote: Counting objects: 100% (186/186), done.
remote: Compressing objects: 100% (101/101), done.
remote: Total 467 (delta 114), reused 137 (delta 85), pack-reused 281
Receiving objects: 100% (467/467), 138.79 KiB | 19.83 MiB/s, done.
Resolving deltas: 100% (185/185), done.
$ ls cks-course-environment/
README.md  cluster-setup  course-content
$ cd cks-course-environment/course-content/supply-chain-security/static-analysis/conftest/
$ ls -l
total 8
drwxrwxr-x 3 ubuntu ubuntu 4096 Jan 21 12:24 docker
drwxrwxr-x 3 ubuntu ubuntu 4096 Jan 21 12:24 kubernetes
$ cd kubernetes/
$ ls -l
total 12
-rw-rw-r-- 1 ubuntu ubuntu  386 Jan 21 12:24 deploy.yaml
drwxrwxr-x 2 ubuntu ubuntu 4096 Jan 21 12:24 policy
-rwxrwxr-x 1 ubuntu ubuntu   77 Jan 21 12:24 run.sh
```

サンプルの中身を確認する。

```deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test
    spec:
      containers:
        - image: httpd
          name: httpd
          resources: {}
status: {}
```

```policy/deployment.rego
# from https://www.conftest.dev
package main

deny[msg] {
  input.kind = "Deployment"
  not input.spec.template.spec.securityContext.runAsNonRoot = true
  msg = "Containers must not run as root"
}

deny[msg] {
  input.kind = "Deployment"
  not input.spec.selector.matchLabels.app
  msg = "Containers must provide app label for pod selectors"
}
```

```run.sh
docker run --rm -v $(pwd):/project openpolicyagent/conftest test deploy.yaml
```

shell scriptを実行する。

```
$ sudo ./run.sh
Unable to find image 'openpolicyagent/conftest:latest' locally
latest: Pulling from openpolicyagent/conftest
c158987b0551: Pull complete
c46e96724b2d: Pull complete
7499f410484d: Pull complete
128b73c106c4: Pull complete
d37d968d2957: Pull complete
Digest: sha256:ecf453c2537bd9d3005df068f35409557061c91807d33b170088ae8caeeee200
Status: Downloaded newer image for openpolicyagent/conftest:latest
FAIL - deploy.yaml - main - Containers must not run as root

2 tests, 1 passed, 0 warnings, 1 failure, 0 exceptions
```

deploy.yamlにSecurityContextで`runAsNonRoot: true`を追記して再度実行する。

```
$ sudo ./run.sh

2 tests, 2 passed, 0 warnings, 0 failures, 0 exceptions
```