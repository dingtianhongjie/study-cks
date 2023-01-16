# マニュアル

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

## OPA

OPAをインストールする。
```
$ kubectl create -f https://raw.githubusercontent.com/killer-sh/cks-course-environment/master/course-content/opa/gatekeeper.yaml
namespace/gatekeeper-system created
resourcequota/gatekeeper-critical-pods created
customresourcedefinition.apiextensions.k8s.io/configs.config.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constraintpodstatuses.status.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constrainttemplatepodstatuses.status.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constrainttemplates.templates.gatekeeper.sh created
serviceaccount/gatekeeper-admin created
role.rbac.authorization.k8s.io/gatekeeper-manager-role created
clusterrole.rbac.authorization.k8s.io/gatekeeper-manager-role created
rolebinding.rbac.authorization.k8s.io/gatekeeper-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/gatekeeper-manager-rolebinding created
secret/gatekeeper-webhook-server-cert created
service/gatekeeper-webhook-service created
Warning: spec.template.metadata.annotations[container.seccomp.security.alpha.kubernetes.io/manager]: deprecated since v1.19, non-functional in a future release; use the "seccompProfile" field instead
deployment.apps/gatekeeper-audit created
deployment.apps/gatekeeper-controller-manager created
validatingwebhookconfiguration.admissionregistration.k8s.io/gatekeeper-validating-webhook-configuration created
```

Podを作成する際に、指定したレジストリのみ使用できるようにする。

```template.yaml
apiVersion: templates.gatekeeper.sh/v1beta1
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
$ kubectl get constrainttemplate
NAME               AGE
k8strustedimages   57s
$ kubectl get k8strustedimages
NAME                 AGE
pod-trusted-images   63s
```

```
$ kubectl describe k8strustedimages pod-trusted-images
Name:         pod-trusted-images
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  constraints.gatekeeper.sh/v1beta1
Kind:         K8sTrustedImages
Metadata:
  Creation Timestamp:  2023-01-16T13:27:31Z
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
    Time:         2023-01-16T13:27:31Z
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
    Time:            2023-01-16T13:29:21Z
  Resource Version:  40886
  UID:               687a61f9-2a22-40c4-8f44-88da5f5ddc21
Spec:
  Match:
    Kinds:
      API Groups:

      Kinds:
        Pod
Status:
  Audit Timestamp:  2023-01-16T13:29:20Z
  By Pod:
    Constraint UID:       687a61f9-2a22-40c4-8f44-88da5f5ddc21
    Enforced:             true
    Id:                   gatekeeper-audit-649cbfdc8f-k8zln
    Observed Generation:  1
    Operations:
      audit
      status
    Constraint UID:       687a61f9-2a22-40c4-8f44-88da5f5ddc21
    Enforced:             true
    Id:                   gatekeeper-controller-manager-57575f6fc7-6pcj4
    Observed Generation:  1
    Operations:
      webhook
    Constraint UID:       687a61f9-2a22-40c4-8f44-88da5f5ddc21
    Enforced:             true
    Id:                   gatekeeper-controller-manager-57575f6fc7-vtj6v
    Observed Generation:  1
    Operations:
      webhook
    Constraint UID:       687a61f9-2a22-40c4-8f44-88da5f5ddc21
    Enforced:             true
    Id:                   gatekeeper-controller-manager-57575f6fc7-zglp4
    Observed Generation:  1
    Operations:
      webhook
  Total Violations:  4
  Violations:
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                gatekeeper-audit-649cbfdc8f-k8zln
    Namespace:           gatekeeper-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                gatekeeper-controller-manager-57575f6fc7-6pcj4
    Namespace:           gatekeeper-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                gatekeeper-controller-manager-57575f6fc7-vtj6v
    Namespace:           gatekeeper-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                gatekeeper-controller-manager-57575f6fc7-zglp4
    Namespace:           gatekeeper-system
Events:                  <none>
```
