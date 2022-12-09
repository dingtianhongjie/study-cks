# マニュアル
https://kubernetes.io/ja/docs/reference/access-authn-authz/rbac/
https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/


# 学習ログ
## namespace resouceとnon-namespace resource

### namespace resouce
namespace resouce = Role/Rolebindingでアクセス制御する。（namespace単位）
ClusterRole/ClusterRoleBindingsでもアクセス制御できるが、クラスタ全体での制御となる。

```
$ kubectl api-resources --namespaced=true
NAME                        SHORTNAMES   APIVERSION                     NAMESPACED   KIND
bindings                                 v1                             true         Binding
configmaps                  cm           v1                             true         ConfigMap
endpoints                   ep           v1                             true         Endpoints
events                      ev           v1                             true         Event
limitranges                 limits       v1                             true         LimitRange
persistentvolumeclaims      pvc          v1                             true         PersistentVolumeClaim
pods                        po           v1                             true         Pod
podtemplates                             v1                             true         PodTemplate
replicationcontrollers      rc           v1                             true         ReplicationController
resourcequotas              quota        v1                             true         ResourceQuota
secrets                                  v1                             true         Secret
serviceaccounts             sa           v1                             true         ServiceAccount
services                    svc          v1                             true         Service
controllerrevisions                      apps/v1                        true         ControllerRevision
daemonsets                  ds           apps/v1                        true         DaemonSet
deployments                 deploy       apps/v1                        true         Deployment
replicasets                 rs           apps/v1                        true         ReplicaSet
statefulsets                sts          apps/v1                        true         StatefulSet
localsubjectaccessreviews                authorization.k8s.io/v1        true         LocalSubjectAccessReview
horizontalpodautoscalers    hpa          autoscaling/v2                 true         HorizontalPodAutoscaler
cronjobs                    cj           batch/v1                       true         CronJob
jobs                                     batch/v1                       true         Job
leases                                   coordination.k8s.io/v1         true         Lease
networkpolicies                          crd.projectcalico.org/v1       true         NetworkPolicy
networksets                              crd.projectcalico.org/v1       true         NetworkSet
endpointslices                           discovery.k8s.io/v1            true         EndpointSlice
events                      ev           events.k8s.io/v1               true         Event
ingresses                   ing          networking.k8s.io/v1           true         Ingress
networkpolicies             netpol       networking.k8s.io/v1           true         NetworkPolicy
poddisruptionbudgets        pdb          policy/v1                      true         PodDisruptionBudget
rolebindings                             rbac.authorization.k8s.io/v1   true         RoleBinding
roles                                    rbac.authorization.k8s.io/v1   true         Role
csistoragecapacities                     storage.k8s.io/v1beta1         true         CSIStorageCapacity
```
### non-namespace resouce
non-namespace resouce = ClusterRole/ClusterRoleBindingsでアクセス制御する。
```
$ kubectl api-resources --namespaced=false
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
componentstatuses                 cs           v1                                     false        ComponentStatus
namespaces                        ns           v1                                     false        Namespace
nodes                             no           v1                                     false        Node
persistentvolumes                 pv           v1                                     false        PersistentVolume
mutatingwebhookconfigurations                  admissionregistration.k8s.io/v1        false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io/v1        false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1                false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io/v1              false        APIService
tokenreviews                                   authentication.k8s.io/v1               false        TokenReview
selfsubjectaccessreviews                       authorization.k8s.io/v1                false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io/v1                false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io/v1                false        SubjectAccessReview
certificatesigningrequests        csr          certificates.k8s.io/v1                 false        CertificateSigningRequest
bgpconfigurations                              crd.projectcalico.org/v1               false        BGPConfiguration
bgppeers                                       crd.projectcalico.org/v1               false        BGPPeer
blockaffinities                                crd.projectcalico.org/v1               false        BlockAffinity
caliconodestatuses                             crd.projectcalico.org/v1               false        CalicoNodeStatus
clusterinformations                            crd.projectcalico.org/v1               false        ClusterInformation
felixconfigurations                            crd.projectcalico.org/v1               false        FelixConfiguration
globalnetworkpolicies                          crd.projectcalico.org/v1               false        GlobalNetworkPolicy
globalnetworksets                              crd.projectcalico.org/v1               false        GlobalNetworkSet
hostendpoints                                  crd.projectcalico.org/v1               false        HostEndpoint
ipamblocks                                     crd.projectcalico.org/v1               false        IPAMBlock
ipamconfigs                                    crd.projectcalico.org/v1               false        IPAMConfig
ipamhandles                                    crd.projectcalico.org/v1               false        IPAMHandle
ippools                                        crd.projectcalico.org/v1               false        IPPool
ipreservations                                 crd.projectcalico.org/v1               false        IPReservation
kubecontrollersconfigurations                  crd.projectcalico.org/v1               false        KubeControllersConfiguration
flowschemas                                    flowcontrol.apiserver.k8s.io/v1beta2   false        FlowSchema
prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1beta2   false        PriorityLevelConfiguration
ingressclasses                                 networking.k8s.io/v1                   false        IngressClass
runtimeclasses                                 node.k8s.io/v1                         false        RuntimeClass
podsecuritypolicies               psp          policy/v1beta1                         false        PodSecurityPolicy
clusterrolebindings                            rbac.authorization.k8s.io/v1           false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io/v1           false        ClusterRole
priorityclasses                   pc           scheduling.k8s.io/v1                   false        PriorityClass
csidrivers                                     storage.k8s.io/v1                      false        CSIDriver
csinodes                                       storage.k8s.io/v1                      false        CSINode
storageclasses                    sc           storage.k8s.io/v1                      false        StorageClass
volumeattachments                              storage.k8s.io/v1                      false        VolumeAttachment
```

## Role/Rolebindings

namespaceを2つ作る。
```
$ kubectl create ns red
namespace/red created
$ kubectl create ns blue
namespace/blue created
```

roleを作成する。
```
$ kubectl -n red create role secret-mgr --verb=get --resource=secret
role.rbac.authorization.k8s.io/secret-mgr created
```

rolebindingを作成する。
```
$ kubectl -n red create rolebinding secret-mgr --role=secret-mgr --user=jane
rolebinding.rbac.authorization.k8s.io/secret-mgr created
```

namespace blueにもroleとrolebindingを作成する。
```
$ kubectl -n blue create role secret-mgr --verb=get,list --resource=secret
role.rbac.authorization.k8s.io/secret-mgr created
$ kubectl -n blue create rolebinding secret-mgr --role=secret-mgr --user=jane
rolebinding.rbac.authorization.k8s.io/secret-mgr created
```

### 確認
```
$ kubectl -n red auth can-i get secret --as jane
yes
$ kubectl -n red auth can-i get secret --as mike
no
$ kubectl -n red auth can-i list secret --as jane
no
$ kubectl -n red auth can-i get pods --as jane
no
```

```
$ kubectl -n blue auth can-i get secret --as jane
yes
$ kubectl -n blue auth can-i list secret --as jane
yes
$ kubectl -n blue auth can-i list pod --as jane
no
$ kubectl -n blue auth can-i list secret --as mike
no
```

## clusterrole/clusterrolebinding

clusterroleとclusterrolebindgを作成する。
```
$ kubectl create clusterrole deploy-deleter --verb=delete --resource=deployment
clusterrole.rbac.authorization.k8s.io/deploy-deleter created
$ kubectl create clusterrolebinding deploy-deleter --clusterrole=deploy-deleter --user=jane
clusterrolebinding.rbac.authorization.k8s.io/deploy-deleter created
```

作成したClusterroleを使用して、namespace単位でアクセス制御する。

```
$ kubectl -n red create rolebinding red-deploy-deleter --clusterrole=deploy-deleter --user=jim
rolebinding.rbac.authorization.k8s.io/red-deploy-deleter created
```

### 確認
```
$ kubectl auth can-i delete deployment --as jane
yes
$ kubectl auth can-i delete deployment --as jane -n default
yes
$ kubectl auth can-i delete deployment --as jane -n red
yes
$ kubectl auth can-i delete deployment --as jane -n blue
yes
$ kubectl auth can-i delete deployment --as jane -A
yes
$ kubectl auth can-i delete pod --as jane
no
$ kubectl auth can-i list deployment --as jane -A
no
```

```
$ kubectl auth can-i delete deployment --as jim
no
$ kubectl auth can-i delete deployment --as jim -A
no
$ kubectl auth can-i delete deployment --as jim -n red
yes
$ kubectl auth can-i delete pod --as jim -n red
no
$ kubectl auth can-i list deployment --as jim -n red
no
```

## Certificate Signing Requests

Keyを作成する。
```
$ openssl genrsa -out jane.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
.............................................................................+++++
...........+++++
e is 65537 (0x010001)
```

Singning Requestを作成する。
ここでは、Common NameにJaneだけ指定する。
```
$ openssl req -new -key jane.key -out jane.csr # only set Common Name = jane
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
Common Name (e.g. server FQDN or YOUR name) []:jane
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

作成したCSRファイルをbase64でエンコードする。

```
$ cat jane.csr | base64 -w 0
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxclJnalFGbDl4ZzBHTHdTclZ4VHdrNnBNdXVxd2k0OWZmQ2Q2d0dTZy9RTE9wUjBHazVzaHYKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
```

マニュアルを参考に以下のマニフェストを作成してapplyする。

```csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx0tCg==
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 2592000
  usages:
  - client auth
```

作成した直後はPendingになっている。

```
$ kubectl apply -f csr.yaml
certificatesigningrequest.certificates.k8s.io/jane created
$ kubectl get csr
NAME   AGE   SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
jane   31s   kubernetes.io/kube-apiserver-client   kubernetes-admin   30d                 Pending
```

これをApproveする。
```
$ kubectl certificate approve jane
certificatesigningrequest.certificates.k8s.io/jane approved
$ kubectl get csr
NAME   AGE    SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
jane   109s   kubernetes.io/kube-apiserver-client   kubernetes-admin   30d                 Approved,Issued
```

作成したcsrをyaml形式で確認する。

```
$ kubectl get csr jane -o yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"certificates.k8s.io/v1","kind":"CertificateSigningRequest","metadata":{"annotations":{},"name":"jane"},"spec":{"expirationSeconds":2592000,"request":"LS0tLS1CRUdJTiBUVxxxxxxxxxxxxxxxxxxxxxxxxxxxxxTVC0tLS0tCg==","signerName":"kubernetes.io/kube-apiserver-client","usages":["client auth"]}}
  creationTimestamp: "2022-12-08T13:16:24Z"
  name: jane
  resourceVersion: "73035"
  uid: cfc4eb25-f5cd-4304-ae57-9dbb0c26d3b3
spec:
  expirationSeconds: 2592000
  groups:
  - system:masters
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNxxxxxxxxxxxxxxxxxxxxxxxxxxxVUVTVC0tLS0tCg==
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
  username: kubernetes-admin
status:
  certificate: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxPVRCBDRVJUSUZJQ0FURS0tLS0tCg==
  conditions:
  - lastTransitionTime: "2022-12-08T13:18:10Z"
    lastUpdateTime: "2022-12-08T13:18:10Z"
    message: This CSR was approved by kubectl certificate approve.
    reason: KubectlApprove
    status: "True"
    type: Approved
```

`status` → `certificate`をコピーしてデコードする。

```
$ echo LS0tLS1CRUdJTiBDRVJUSUZJQ0FxxxxxxxxxxxxxxxxxxxxxxURS0tLS0tCg== | base64 -d > jane.crt
```

kubeconfig に user janeを追加する。

```
$ kubectl config set-credentials jane --client-key=jane.key --client-certificate=jane.crt
User "jane" set.
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.0.0.218:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: jane
  user:
    client-certificate: /home/ubuntu/12/jane.crt
    client-key: /home/ubuntu/12/jane.key
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

contextに user janeを追加する。

```
$ kubectl config set-context jane --user=jane --cluster=kubernetes
Context "jane" created.
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.0.0.218:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: jane
  name: jane
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: jane
  user:
    client-certificate: /home/ubuntu/12/jane.crt
    client-key: /home/ubuntu/12/jane.key
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
$ kubectl config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
          jane                          kubernetes   jane
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
```

context を janeにスイッチする。

```
$ kubectl config use-context jane
Switched to context "jane".
$ kubectl config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         jane                          kubernetes   jane
          kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
```

権限を確認する。

```
$ kubectl get node
Error from server (Forbidden): nodes is forbidden: User "jane" cannot list resource "nodes" in API group "" at the cluster scope
$ kubectl get ns
Error from server (Forbidden): namespaces is forbidden: User "jane" cannot list resource "namespaces" in API group "" at the cluster scope
$ kubectl -n blue get secret
NAME                  TYPE                                  DATA   AGE
default-token-b8xlr   kubernetes.io/service-account-token   3      47h
```

RBACで指定した権限が有効になっている。
