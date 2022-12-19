# マニュアル
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

# 学習ログ
## Control planeのアップグレード

```
$ kubectl get node
NAME              STATUS   ROLES           AGE     VERSION
master-20221214   Ready    control-plane   4d23h   v1.25.4
worker-20221214   Ready    <none>          4d23h   v1.25.4
```

### kubeadmのアップグレード

```
$ sudo apt update
Get:1 https://download.docker.com/linux/ubuntu focal InRelease [57.7 kB]
Hit:2 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Hit:3 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal InRelease
Get:4 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:5 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:6 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Fetched 394 kB in 1s (548 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
18 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

```
$ apt-cache madison kubeadm |grep 1.25
   kubeadm |  1.25.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
```

今回は1.25.4から1.25.5にアップグレードする。

```
$ sudo  apt-mark unhold kubeadm
Canceled hold on kubeadm.
$ sudo apt-get update && apt-get install -y kubeadm=1.25.5-00
Hit:1 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:2 https://download.docker.com/linux/ubuntu focal InRelease
Hit:4 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal InRelease
Hit:5 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:3 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Hit:6 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-backports InRelease
Reading package lists... Done
E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?
```

失敗したので、ここからrootにする。

```
$ sudo su -
# apt-get update && apt-get install -y kubeadm=1.25.5-00
Hit:1 https://download.docker.com/linux/ubuntu focal InRelease
Hit:3 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal InRelease
Hit:4 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:5 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:6 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-backports InRelease
Hit:2 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 17 not upgraded.
Need to get 9229 kB of archives.
After this operation, 8192 B of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.25.5-00 [9229 kB]
Fetched 9229 kB in 0s (27.4 MB/s)
(Reading database ... 144483 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.25.5-00_amd64.deb ...
Unpacking kubeadm (1.25.5-00) over (1.25.4-00) ...
Setting up kubeadm (1.25.5-00) ...
# apt-mark hold kubeadm
kubeadm set on hold.
```

kubeadmのバージョンを確認する。

```
# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"25", GitVersion:"v1.25.5", GitCommit:"804d6167111f6858541cef440ccc53887fbbc96a", GitTreeState:"clean", BuildDate:"2022-12-08T10:13:29Z", GoVersion:"go1.19.4", Compiler:"gc", Platform:"linux/amd64"}
```
```
# kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.25.5
[upgrade/versions] kubeadm version: v1.25.5
I1219 13:18:17.722041  123756 version.go:256] remote version is much newer: v1.26.0; falling back to: stable-1.25
[upgrade/versions] Target version: v1.25.5
[upgrade/versions] Latest version in the v1.25 series: v1.25.5
```

あれ？ログが思ってたのと違う。v1.26があるからかな？

気にせず続ける。

```
# kubeadm upgrade apply v1.25.5
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.25.5"
[upgrade/versions] Cluster version: v1.25.5
[upgrade/versions] kubeadm version: v1.25.5
[upgrade] Are you sure you want to proceed? [y/N]: y
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet connection
[upgrade/prepull] You can also perform this action in beforehand using 'kubeadm config images pull'
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.25.5" (timeout: 5m0s)...
[upgrade/etcd] Upgrading to TLS for etcd
[upgrade/staticpods] Preparing for "etcd" upgrade
[upgrade/staticpods] Renewing etcd-server certificate
[upgrade/staticpods] Renewing etcd-peer certificate
[upgrade/staticpods] Renewing etcd-healthcheck-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/etcd.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2022-12-19-13-19-59/etcd.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
E1219 13:20:52.604139  124343 request.go:1058] Unexpected error when reading response body: context deadline exceeded (Client.Timeout or context cancellation while reading body)
[apiclient] Found 1 Pods for label selector component=etcd
[upgrade/staticpods] Component "etcd" upgraded successfully!
[upgrade/etcd] Waiting for etcd to become available
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests2873522880"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Current and new manifests of kube-apiserver are equal, skipping upgrade
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Current and new manifests of kube-controller-manager are equal, skipping upgrade
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Current and new manifests of kube-scheduler are equal, skipping upgrade
[upgrade/postupgrade] Removing the old taint &Taint{Key:node-role.kubernetes.io/master,Value:,Effect:NoSchedule,TimeAdded:<nil>,} from all control plane Nodes. After this step only the &Taint{Key:node-role.kubernetes.io/control-plane,Value:,Effect:NoSchedule,TimeAdded:<nil>,} taint will be present on control plane Nodes.
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.25.5". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```

### kubelet/kubectlのアップグレード

```
$ kubectl drain master-20221214 --ignore-daemonsets
node/master-20221214 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/calico-node-zm8hh, kube-system/kube-proxy-t6c9b
node/master-20221214 drained
$ kubectl get node
NAME              STATUS                     ROLES           AGE   VERSION
master-20221214   Ready,SchedulingDisabled   control-plane   5d    v1.25.4
worker-20221214   Ready                      <none>          5d    v1.25.4
```

```
# apt-mark unhold kubelet kubectl
Canceled hold on kubelet.
Canceled hold on kubectl.
# apt-get update && apt-get install -y kubelet=1.25.5-00 kubectl=1.25.5-00
Hit:1 https://download.docker.com/linux/ubuntu focal InRelease
Hit:2 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal InRelease
Hit:3 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:4 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-backports InRelease
Hit:6 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:5 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 16 not upgraded.
Need to get 29.0 MB of archives.
After this operation, 24.6 kB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubectl amd64 1.25.5-00 [9509 kB]
Get:2 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.25.5-00 [19.5 MB]
Fetched 29.0 MB in 1s (32.8 MB/s)
(Reading database ... 144483 files and directories currently installed.)
Preparing to unpack .../kubectl_1.25.5-00_amd64.deb ...
Unpacking kubectl (1.25.5-00) over (1.25.4-00) ...
Preparing to unpack .../kubelet_1.25.5-00_amd64.deb ...
Unpacking kubelet (1.25.5-00) over (1.25.4-00) ...
Setting up kubectl (1.25.5-00) ...
Setting up kubelet (1.25.5-00) ...
# apt-mark hold kubelet kubectl
kubelet set on hold.
kubectl set on hold.
```

```
# systemctl daemon-reload
# systemctl restart kubelet
# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             mq10-kubeadm.conf
     Active: active (running) since Mon 2022-12-19 13:25:43 UTC; 36s ago
       Docs: https://kubernetes.io/docs/home/
   Main PID: 128024 (kubelet)
      Tasks: 15 (limit: 9444)
     Memory: 33.7M
     CGroup: /system.slice/kubelet.service
             mq128024 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=remote --container-runtime-endpoint>

Dec 19 13:25:44 master-20221214 kubelet[128024]: I1219 13:25:44.679978  128024 reconciler.go:357] "operationExecutor.VerifyControllerAttachedVolume started for volume \"lib-modules\" (UniqueName: \"kubernetes.io/host-path/9a685b25-4ce6->
Dec 19 13:25:44 master-20221214 kubelet[128024]: I1219 13:25:44.680027  128024 reconciler.go:357] "operationExecutor.VerifyControllerAttachedVolume started for volume \"var-run-calico\" (UniqueName: \"kubernetes.io/host-path/9a685b25-4c>
Dec 19 13:25:44 master-20221214 kubelet[128024]: I1219 13:25:44.680047  128024 reconciler.go:169] "Reconciler: start to sync state"
Dec 19 13:25:44 master-20221214 kubelet[128024]: E1219 13:25:44.827843  128024 kubelet.go:1712] "Failed creating a mirror pod for" err="pods \"kube-controller-manager-master-20221214\" already exists" pod="kube-system/kube-controller-ma>
Dec 19 13:25:45 master-20221214 kubelet[128024]: E1219 13:25:45.029268  128024 kubelet.go:1712] "Failed creating a mirror pod for" err="pods \"kube-apiserver-master-20221214\" already exists" pod="kube-system/kube-apiserver-master-20221>
Dec 19 13:25:45 master-20221214 kubelet[128024]: E1219 13:25:45.226331  128024 kubelet.go:1712] "Failed creating a mirror pod for" err="pods \"kube-scheduler-master-20221214\" already exists" pod="kube-system/kube-scheduler-master-20221>
Dec 19 13:25:45 master-20221214 kubelet[128024]: I1219 13:25:45.421236  128024 request.go:682] Waited for 1.078088734s due to client-side throttling, not priority and fairness, request: POST:https://10.0.0.196:6443/api/v1/namespaces/kub>
Dec 19 13:25:45 master-20221214 kubelet[128024]: E1219 13:25:45.426540  128024 kubelet.go:1712] "Failed creating a mirror pod for" err="pods \"etcd-master-20221214\" already exists" pod="kube-system/etcd-master-20221214"
Dec 19 13:25:45 master-20221214 kubelet[128024]: E1219 13:25:45.781340  128024 configmap.go:197] Couldn't get configMap kube-system/kube-proxy: failed to sync configmap cache: timed out waiting for the condition
Dec 19 13:25:45 master-20221214 kubelet[128024]: E1219 13:25:45.781443  128024 nestedpendingoperations.go:348] Operation for "{volumeName:kubernetes.io/configmap/34e3158c-f9c6-4661-8970-7bee5cbe0c8e-kube-proxy podName:34e3158c-f9c6-4661>

```

```
$ kubectl uncordon master-20221214
node/master-20221214 uncordoned
$ kubectl get node
NAME              STATUS   ROLES           AGE   VERSION
master-20221214   Ready    control-plane   5d    v1.25.5
worker-20221214   Ready    <none>          5d    v1.25.4
```

## Worker Nodeのアップグレード
### kubeadmのアップグレード
```
# apt-mark unhold kubeadm
Canceled hold on kubeadm.
# apt-get update && apt-get install -y kubeadm=1.25.5-00
Hit:2 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal InRelease
Get:3 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:4 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:5 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Hit:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Fetched 336 kB in 1s (436 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 16 not upgraded.
Need to get 9229 kB of archives.
After this operation, 8192 B of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.25.5-00 [9229 kB]
Fetched 9229 kB in 0s (33.8 MB/s)
(Reading database ... 144483 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.25.5-00_amd64.deb ...
Unpacking kubeadm (1.25.5-00) over (1.25.4-00) ...
Setting up kubeadm (1.25.5-00) ...
# apt-mark hold kubeadm
kubeadm set on hold.
```

```
# kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
```

### kubelet/kubectlのアップグレード

（Control Planeで実施）
```
$ kubectl drain worker-20221214 --ignore-daemonsets
node/worker-20221214 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/calico-node-k8sd6, kube-system/kube-proxy-58wdb
evicting pod kube-system/coredns-565d847f94-fwvh2
evicting pod kube-system/calico-kube-controllers-798cc86c47-njz4l
evicting pod kube-system/coredns-565d847f94-7vs6g
pod/calico-kube-controllers-798cc86c47-njz4l evicted
pod/coredns-565d847f94-7vs6g evicted
pod/coredns-565d847f94-fwvh2 evicted
node/worker-20221214 drained
```

（ここからWorkerで実施）
```
# apt-mark unhold kubelet kubectl
Canceled hold on kubelet.
Canceled hold on kubectl.
# apt-get update && apt-get install -y kubelet=1.25.5-00 kubectl=1.25.5-00
Hit:1 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:2 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal InRelease
Hit:3 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:5 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-backports InRelease
Hit:4 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 15 not upgraded.
Need to get 29.0 MB of archives.
After this operation, 24.6 kB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubectl amd64 1.25.5-00 [9509 kB]
Get:2 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.25.5-00 [19.5 MB]
Fetched 29.0 MB in 1s (42.8 MB/s)
(Reading database ... 144483 files and directories currently installed.)
Preparing to unpack .../kubectl_1.25.5-00_amd64.deb ...
Unpacking kubectl (1.25.5-00) over (1.25.4-00) ...
Preparing to unpack .../kubelet_1.25.5-00_amd64.deb ...
Unpacking kubelet (1.25.5-00) over (1.25.4-00) ...
Setting up kubectl (1.25.5-00) ...
Setting up kubelet (1.25.5-00) ...
# apt-mark hold kubelet kubectl
kubelet set on hold.
kubectl set on hold.
```

```
# systemctl daemon-reload
# systemctl restart kubelet
# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             mq10-kubeadm.conf
     Active: active (running) since Mon 2022-12-19 13:36:08 UTC; 5s ago
       Docs: https://kubernetes.io/docs/home/
   Main PID: 229225 (kubelet)
      Tasks: 13 (limit: 19106)
     Memory: 29.8M
     CGroup: /system.slice/kubelet.service
             mq229225 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=remote --container-runtime-endpoint>

Dec 19 13:36:10 worker-20221214 kubelet[229225]: I1219 13:36:10.029481  229225 reconciler.go:357] "operationExecutor.VerifyControllerAttachedVolume started for volume \"cni-net-dir\" (UniqueName: \"kubernetes.io/host-path/7cece762-2a1b->
Dec 19 13:36:10 worker-20221214 kubelet[229225]: I1219 13:36:10.029510  229225 reconciler.go:357] "operationExecutor.VerifyControllerAttachedVolume started for volume \"policysync\" (UniqueName: \"kubernetes.io/host-path/7cece762-2a1b-4>
Dec 19 13:36:10 worker-20221214 kubelet[229225]: I1219 13:36:10.029540  229225 reconciler.go:357] "operationExecutor.VerifyControllerAttachedVolume started for volume \"kube-proxy\" (UniqueName: \"kubernetes.io/configmap/7bd2327b-7884-4>
Dec 19 13:36:10 worker-20221214 kubelet[229225]: I1219 13:36:10.029569  229225 reconciler.go:357] "operationExecutor.VerifyControllerAttachedVolume started for volume \"kube-api-access-5z62d\" (UniqueName: \"kubernetes.io/projected/7bd2>
Dec 19 13:36:10 worker-20221214 kubelet[229225]: I1219 13:36:10.029595  229225 reconciler.go:357] "operationExecutor.VerifyControllerAttachedVolume started for volume \"lib-modules\" (UniqueName: \"kubernetes.io/host-path/7cece762-2a1b->
Dec 19 13:36:10 worker-20221214 kubelet[229225]: I1219 13:36:10.029622  229225 reconciler.go:357] "operationExecutor.VerifyControllerAttachedVolume started for volume \"xtables-lock\" (UniqueName: \"kubernetes.io/host-path/7cece762-2a1b>
Dec 19 13:36:10 worker-20221214 kubelet[229225]: I1219 13:36:10.029651  229225 reconciler.go:357] "operationExecutor.VerifyControllerAttachedVolume started for volume \"bpffs\" (UniqueName: \"kubernetes.io/host-path/7cece762-2a1b-4f95-a>
Dec 19 13:36:10 worker-20221214 kubelet[229225]: I1219 13:36:10.029684  229225 reconciler.go:357] "operationExecutor.VerifyControllerAttachedVolume started for volume \"cni-bin-dir\" (UniqueName: \"kubernetes.io/host-path/7cece762-2a1b->
Dec 19 13:36:10 worker-20221214 kubelet[229225]: I1219 13:36:10.029721  229225 reconciler.go:357] "operationExecutor.VerifyControllerAttachedVolume started for volume \"kube-api-access-g7g9g\" (UniqueName: \"kubernetes.io/projected/7cec>
Dec 19 13:36:10 worker-20221214 kubelet[229225]: I1219 13:36:10.029732  229225 reconciler.go:169] "Reconciler: start to sync state"
```

（Control Planeで実施）
```
$ kubectl uncordon worker-20221214
node/worker-20221214 uncordoned
$ kubectl get node
NAME              STATUS   ROLES           AGE   VERSION
master-20221214   Ready    control-plane   5d    v1.25.5
worker-20221214   Ready    <none>          5d    v1.25.5
```