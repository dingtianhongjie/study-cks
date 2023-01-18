# マニュアル

https://open-policy-agent.github.io/gatekeeper/website/docs/install/

# 学習ログ
## Image digest

使用されているContainer ImageとResistoryを確認する。
```
$ kubectl get pod -A -o yaml | grep "image:"
      image: docker.io/calico/kube-controllers:v3.25.0
      image: docker.io/calico/kube-controllers:v3.25.0
      image: docker.io/calico/node:v3.25.0
      image: docker.io/calico/cni:v3.25.0
      image: docker.io/calico/cni:v3.25.0
      image: docker.io/calico/node:v3.25.0
      image: docker.io/calico/node:v3.25.0
      image: docker.io/calico/cni:v3.25.0
      image: docker.io/calico/cni:v3.25.0
      image: docker.io/calico/node:v3.25.0
      image: docker.io/calico/node:v3.25.0
      image: docker.io/calico/cni:v3.25.0
      image: docker.io/calico/cni:v3.25.0
      image: docker.io/calico/node:v3.25.0
      image: docker.io/calico/node:v3.25.0
      image: docker.io/calico/cni:v3.25.0
      image: docker.io/calico/cni:v3.25.0
      image: docker.io/calico/node:v3.25.0
      image: registry.k8s.io/coredns/coredns:v1.9.3
      image: registry.k8s.io/coredns/coredns:v1.9.3
      image: registry.k8s.io/coredns/coredns:v1.9.3
      image: registry.k8s.io/coredns/coredns:v1.9.3
      image: registry.k8s.io/etcd:3.5.5-0
      image: registry.k8s.io/etcd:3.5.5-0
      image: registry.k8s.io/kube-apiserver:v1.25.5
      image: registry.k8s.io/kube-apiserver:v1.25.5
      image: registry.k8s.io/kube-controller-manager:v1.25.5
      image: registry.k8s.io/kube-controller-manager:v1.25.5
      image: registry.k8s.io/kube-proxy:v1.25.5
      image: registry.k8s.io/kube-proxy:v1.25.5
      image: registry.k8s.io/kube-proxy:v1.25.5
      image: registry.k8s.io/kube-proxy:v1.25.5
      image: registry.k8s.io/kube-scheduler:v1.25.5
      image: registry.k8s.io/kube-scheduler:v1.25.5
```

API ServerのimageIDを確認する。

```
$ kubectl get pod -n kube-system kube-apiserver-master-20230115 -o yaml |grep imageID
    imageID: registry.k8s.io/kube-apiserver@sha256:8f848ab4be2f3f29f220a9bb7e9152f77a8185bf2ad65e5204cc7b53aa456527
```

API ServerのマニフェストのimageをこのIDを指定して書き換える。

```
# vim /etc/kubernetes/manifests/kube-apiserver.yaml
```
API Serverが上がってくることを確認する。

## OPA Gatekeeper

Gatekeeperをインストールする。

```
$ kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/master/deploy/gatekeeper.yaml
namespace/gatekeeper-system created
resourcequota/gatekeeper-critical-pods created
customresourcedefinition.apiextensions.k8s.io/assign.mutations.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/assignmetadata.mutations.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/configs.config.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constraintpodstatuses.status.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constrainttemplatepodstatuses.status.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constrainttemplates.templates.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/expansiontemplate.expansion.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/modifyset.mutations.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/mutatorpodstatuses.status.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/providers.externaldata.gatekeeper.sh created
serviceaccount/gatekeeper-admin created
role.rbac.authorization.k8s.io/gatekeeper-manager-role created
clusterrole.rbac.authorization.k8s.io/gatekeeper-manager-role created
rolebinding.rbac.authorization.k8s.io/gatekeeper-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/gatekeeper-manager-rolebinding created
secret/gatekeeper-webhook-server-cert created
service/gatekeeper-webhook-service created
deployment.apps/gatekeeper-audit created
deployment.apps/gatekeeper-controller-manager created
poddisruptionbudget.policy/gatekeeper-controller-manager created
mutatingwebhookconfiguration.admissionregistration.k8s.io/gatekeeper-mutating-webhook-configuration created
validatingwebhookconfiguration.admissionregistration.k8s.io/gatekeeper-validating-webhook-configuration created
```

```
$ kubectl -n gatekeeper-system get all
NAME                                                 READY   STATUS    RESTARTS      AGE
pod/gatekeeper-audit-7f4c69f44c-hgc89                1/1     Running   2 (24s ago)   28s
pod/gatekeeper-controller-manager-6ff6d5cf86-2pv2f   1/1     Running   0             28s
pod/gatekeeper-controller-manager-6ff6d5cf86-6zp5s   1/1     Running   0             28s
pod/gatekeeper-controller-manager-6ff6d5cf86-nbnsq   1/1     Running   0             28s

NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/gatekeeper-webhook-service   ClusterIP   10.102.234.25   <none>        443/TCP   28s

NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/gatekeeper-audit                1/1     1            1           28s
deployment.apps/gatekeeper-controller-manager   3/3     3            3           28s

NAME                                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/gatekeeper-audit-7f4c69f44c                1         1         1       28s
replicaset.apps/gatekeeper-controller-manager-6ff6d5cf86   3         3         3       28s
```

Podを作成する際に、指定したレジストリのみ使用できるようにする。

```template.yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8strustedimages
spec:
  crd:
    spec:
      names:
        kind: K8sTrustedImages
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8strustedimages
        violation[{"msg": msg}] {
          image := input.review.object.spec.containers[_].image
          not startswith(image, "docker.io/")
          not startswith(image, "registry.k8s.io/")
          not startswith(image, "openpolicyagent/")
          msg := "not trusted image!"
        }
```

```constraint.yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sTrustedImages
metadata:
  name: pod-trusted-images
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
```

```
$ kubectl apply -f template.yaml
constrainttemplate.templates.gatekeeper.sh/k8strustedimages created
$ kubectl apply -f constraint.yaml
k8strustedimages.constraints.gatekeeper.sh/pod-trusted-images created
```

```
$ kubectl get constrainttemplate
NAME               AGE
k8strustedimages   22s
$ kubectl get k8strustedimages.constraints.gatekeeper.sh
NAME                 ENFORCEMENT-ACTION   TOTAL-VIOLATIONS
pod-trusted-images
```

確認する。

```
$ kubectl run good --image=docker.io/nginx
pod/good created
$ kubectl run bad --image=nginx
pod/bad created
```

レジストリを指定しなくてもPodが作れてしまう。。

```
$ kubectl get k8strustedimages.constraints.gatekeeper.sh
NAME                 ENFORCEMENT-ACTION   TOTAL-VIOLATIONS
pod-trusted-images                        1
$ kubectl describe k8strustedimages.constraints.gatekeeper.sh pod-trusted-images
Name:         pod-trusted-images
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  constraints.gatekeeper.sh/v1beta1
Kind:         K8sTrustedImages
Metadata:
  Creation Timestamp:  2023-01-18T13:10:02Z
  Generation:          1
  Managed Fields:
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        .:
        f:match:
          .:
          f:kinds:
    Manager:      kubectl-client-side-apply
    Operation:    Update
    Time:         2023-01-18T13:10:01Z
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        .:
        f:auditTimestamp:
        f:byPod:
        f:totalViolations:
        f:violations:
    Manager:         gatekeeper
    Operation:       Update
    Subresource:     status
    Time:            2023-01-18T13:12:38Z
  Resource Version:  92879
  UID:               0dcf091d-7d0a-4ac3-a0c4-1d64ff1d8c39
Spec:
  Match:
    Kinds:
      API Groups:

      Kinds:
        Pod
Status:
  Audit Timestamp:  2023-01-18T13:12:36Z
  By Pod:
    Constraint UID:       0dcf091d-7d0a-4ac3-a0c4-1d64ff1d8c39
    Enforced:             true
    Id:                   gatekeeper-audit-7f4c69f44c-hgc89
    Observed Generation:  1
    Operations:
      audit
      mutation-status
      status
    Constraint UID:       0dcf091d-7d0a-4ac3-a0c4-1d64ff1d8c39
    Enforced:             true
    Id:                   gatekeeper-controller-manager-6ff6d5cf86-2pv2f
    Observed Generation:  1
    Operations:
      mutation-webhook
      webhook
    Constraint UID:       0dcf091d-7d0a-4ac3-a0c4-1d64ff1d8c39
    Enforced:             true
    Id:                   gatekeeper-controller-manager-6ff6d5cf86-6zp5s
    Observed Generation:  1
    Operations:
      mutation-webhook
      webhook
    Constraint UID:       0dcf091d-7d0a-4ac3-a0c4-1d64ff1d8c39
    Enforced:             true
    Id:                   gatekeeper-controller-manager-6ff6d5cf86-nbnsq
    Observed Generation:  1
    Operations:
      mutation-webhook
      webhook
  Total Violations:  1
  Violations:
    Enforcement Action:  deny
    Group:
    Kind:                Pod
    Message:             not trusted image!
    Name:                bad
    Namespace:           default
    Version:             v1
Events:                  <none>
```

Violationsとしてbadが拒否されているとなっているが・・・

```
$ kubectl get pod
NAME   READY   STATUS    RESTARTS   AGE
bad    1/1     Running   0          95s
good   1/1     Running   0          3m22s
```
Podはデプロイされている？？
んー。。。