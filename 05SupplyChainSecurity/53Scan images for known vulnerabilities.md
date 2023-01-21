# マニュアル
https://github.com/aquasecurity/trivy#docker

https://aquasecurity.github.io/trivy/v0.36/getting-started/installation/

# 学習ログ
## Trivy
### Docker

```
$ sudo docker run ghcr.io/aquasecurity/trivy:0.36.1
Unable to find image 'ghcr.io/aquasecurity/trivy:0.36.1' locally
0.36.1: Pulling from aquasecurity/trivy
c158987b0551: Already exists
a9dc943b72f7: Pull complete
2f5203fcdb82: Pull complete
02e56db24b04: Pull complete
Digest: sha256:fcd4eddc8082be2d7c929cb07c989d62d2d50669513b8a0889116b40feab435f
Status: Downloaded newer image for ghcr.io/aquasecurity/trivy:0.36.1
Scanner for vulnerabilities in container images, file systems, and Git repositories, as well as for configuration issues and hard-coded secrets

Usage:
  trivy [global flags] command [flags] target
  trivy [command]

Examples:
  # Scan a container image
  $ trivy image python:3.4-alpine

  # Scan a container image from a tar archive
  $ trivy image --input ruby-3.1.tar

  # Scan local filesystem
  $ trivy fs .

  # Run in server mode
  $ trivy server

Available Commands:
  aws         [EXPERIMENTAL] Scan AWS account
  config      Scan config files for misconfigurations
  filesystem  Scan local filesystem
  help        Help about any command
  image       Scan a container image
  kubernetes  [EXPERIMENTAL] Scan kubernetes cluster
  module      Manage modules
  plugin      Manage plugins
  repository  Scan a remote repository
  rootfs      Scan rootfs
  sbom        Scan SBOM for vulnerabilities
  server      Server mode
  version     Print the version
  vm          [EXPERIMENTAL] Scan a virtual machine image

Flags:
      --cache-dir string          cache directory (default "/root/.cache/trivy")
  -c, --config string             config path (default "trivy.yaml")
  -d, --debug                     debug mode
  -f, --format string             version format (json)
      --generate-default-config   write the default config to trivy-default.yaml
  -h, --help                      help for trivy
      --insecure                  allow insecure server connections when using TLS
  -q, --quiet                     suppress progress bar and log output
      --timeout duration          timeout (default 5m0s)
  -v, --version                   show version

Use "trivy [command] --help" for more information about a command.
```

コンテナイメージを指定して実行すると、そのイメージの脆弱性が診断される

```
$ sudo docker run ghcr.io/aquasecurity/trivy:0.36.1 image nginx
2023-01-21T13:12:16.777Z        INFO    Need to update DB
2023-01-21T13:12:16.777Z        INFO    DB Repository: ghcr.io/aquasecurity/trivy-db
2023-01-21T13:12:16.777Z        INFO    Downloading DB...
21.75 MiB / 36.16 MiB [------------------------------------>________________________] 60.15% ? p/s ?36.16 MiB / 36.16 MiB [----------------------------------------------------------->] 100.00% ? p/s ?36.16 MiB / 36.16 MiB [----------------------------------------------------------->] 100.00% ? p/s ?36.16 MiB / 36.16 MiB [---------------------------------------------->] 100.00% 24.04 MiB p/s ETA 0s36.16 MiB / 36.16 MiB [---------------------------------------------->] 100.00% 24.04 MiB p/s ETA 0s36.16 MiB / 36.16 MiB [---------------------------------------------->] 100.00% 24.04 MiB p/s ETA 0s36.16 MiB / 36.16 MiB [---------------------------------------------->] 100.00% 22.49 MiB p/s ETA 0s36.16 MiB / 36.16 MiB [---------------------------------------------->] 100.00% 22.49 MiB p/s ETA 0s36.16 MiB / 36.16 MiB [---------------------------------------------->] 100.00% 22.49 MiB p/s ETA 0s36.16 MiB / 36.16 MiB [-------------------------------------------------] 100.00% 21.18 MiB p/s 1.9s2023-01-21T13:12:19.019Z    INFO    Vulnerability scanning is enabled
2023-01-21T13:12:19.019Z        INFO    Secret scanning is enabled
2023-01-21T13:12:19.019Z        INFO    If your scanning is slow, please try '--security-checks vuln' to disable secret scanning
2023-01-21T13:12:19.019Z        INFO    Please see also https://aquasecurity.github.io/trivy/v0.36/docs/secret/scanning/#recommendation for faster secret detection
2023-01-21T13:12:24.746Z        INFO    Detected OS: debian
2023-01-21T13:12:24.746Z        INFO    Detecting Debian vulnerabilities...
2023-01-21T13:12:24.768Z        INFO    Number of language-specific files: 0

nginx (debian 11.6)
===================
Total: 145 (UNKNOWN: 0, LOW: 93, MEDIUM: 33, HIGH: 16, CRITICAL: 3)

・・・
```

### Install

```
$ sudo apt-get install wget apt-transport-https gnupg lsb-release
Reading package lists... Done
Building dependency tree
Reading state information... Done
lsb-release is already the newest version (11.1.0ubuntu2).
lsb-release set to manually installed.
gnupg is already the newest version (2.2.19-3ubuntu2.2).
gnupg set to manually installed.
wget is already the newest version (1.20.3-1ubuntu2).
wget set to manually installed.
apt-transport-https is already the newest version (2.0.9).
0 upgraded, 0 newly installed, 0 to remove and 32 not upgraded.
$ wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
$ echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb focal main
$ sudo apt-get update
Hit:1 https://download.docker.com/linux/ubuntu focal InRelease
Hit:2 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal InRelease
Hit:3 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:4 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:5 http://iad-ad-3.clouds.archive.ubuntu.com/ubuntu focal-backports InRelease
Get:6 https://aquasecurity.github.io/trivy-repo/deb focal InRelease [3053 B]
Hit:7 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Get:8 https://aquasecurity.github.io/trivy-repo/deb focal/main amd64 Packages [376 B]
Fetched 3429 B in 0s (7104 B/s)
Reading package lists... Done
ubuntu@master-20230115:~/23$ sudo apt-get install trivy
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  trivy
0 upgraded, 1 newly installed, 0 to remove and 32 not upgraded.
Need to get 48.6 MB of archives.
After this operation, 182 MB of additional disk space will be used.
Get:1 https://aquasecurity.github.io/trivy-repo/deb focal/main amd64 trivy amd64 0.36.1 [48.6 MB]
Fetched 48.6 MB in 1s (79.8 MB/s)
Selecting previously unselected package trivy.
(Reading database ... 180166 files and directories currently installed.)
Preparing to unpack .../trivy_0.36.1_amd64.deb ...
Unpacking trivy (0.36.1) ...
Setting up trivy (0.36.1) ...
```

trivyコマンドが使えるようになる。

```
$ trivy image nginx
2023-01-21T13:26:28.850Z        INFO    Need to update DB
2023-01-21T13:26:28.850Z        INFO    DB Repository: ghcr.io/aquasecurity/trivy-db
2023-01-21T13:26:28.850Z        INFO    Downloading DB...
36.16 MiB / 36.16 MiB [------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------] 100.00% 26.70 MiB p/s 1.6s
2023-01-21T13:26:30.703Z        INFO    Vulnerability scanning is enabled
2023-01-21T13:26:30.703Z        INFO    Secret scanning is enabled
2023-01-21T13:26:30.703Z        INFO    If your scanning is slow, please try '--security-checks vuln' to disable secret scanning
2023-01-21T13:26:30.703Z        INFO    Please see also https://aquasecurity.github.io/trivy/v0.36/docs/secret/scanning/#recommendation for faster secret detection
2023-01-21T13:26:36.126Z        INFO    Detected OS: debian
2023-01-21T13:26:36.127Z        INFO    Detecting Debian vulnerabilities...
2023-01-21T13:26:36.147Z        INFO    Number of language-specific files: 0

nginx (debian 11.6)

Total: 145 (UNKNOWN: 0, LOW: 93, MEDIUM: 33, HIGH: 16, CRITICAL: 3)
・・・
```

```
$ trivy k8s --report summary cluster
161 / 161 [-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------] 100.00% 1 p/s

Summary Report for kubernetes-admin@kubernetes


Workload Assessment
lqqqqqqqqqqqqqwqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqwqqqqqqqqqqqqqqqqqqqqqqwqqqqqqqqqqqqqqqqqqqqqwqqqqqqqqqqqqqqqqqqqk
x  Namespace  x                   Resource                   x   Vulnerabilities    x  Misconfigurations  x      Secrets      x
x             x                                              tqqqwqqqqwqqqqwqqqqwqqqnqqqwqqqwqqqqwqqqqwqqqnqqqwqqqwqqqwqqqwqqqu
x             x                                              x C x H  x M  x L  x U x C x H x M  x L  x U x C x H x M x L x U x
tqqqqqqqqqqqqqnqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqnqqqnqqqqnqqqqnqqqqnqqqnqqqnqqqnqqqqnqqqqnqqqnqqqnqqqnqqqnqqqnqqqu
x kube-system x DaemonSet/calico-node                        x 8 x    x 40 x 18 x   x   x 5 x 10 x 39 x   x   x   x   x   x   x
x kube-system x DaemonSet/kube-proxy                         x   x 1  x 3  x 22 x   x   x 2 x 4  x 10 x   x   x   x   x   x   x
x kube-system x Deployment/coredns                           x 1 x 3  x 2  x 1  x   x   x   x 3  x 5  x   x   x   x   x   x   x
x kube-system x Pod/kube-scheduler-master-20230115           x   x    x 1  x    x   x   x 1 x 3  x 8  x   x   x   x   x   x   x
x kube-system x ConfigMap/extension-apiserver-authentication x   x    x    x    x   x   x 1 x    x    x   x   x   x   x   x   x
x kube-system x Pod/kube-apiserver-master-20230115           x   x    x 2  x 1  x   x   x 1 x 3  x 9  x   x   x   x   x   x   x
x kube-system x Service/kube-dns                             x   x    x    x    x   x   x   x 1  x    x   x   x   x   x   x   x
x kube-system x Pod/etcd-master-20230115                     x   x 16 x 8  x    x   x   x 1 x 3  x 7  x   x   x   x   x   x   x
x kube-system x Pod/kube-controller-manager-master-20230115  x   x    x 2  x 1  x   x   x 1 x 3  x 8  x   x   x   x   x   x   x
x kube-system x Deployment/calico-kube-controllers           x 1 x    x    x    x   x   x   x 3  x 10 x   x   x   x   x   x   x
x default     x Service/kubernetes                           x   x    x    x    x   x   x   x    x 1  x   x   x   x   x   x   x
x default     x ConfigMap/kube-root-ca.crt                   x   x    x    x    x   x   x   x    x 1  x   x   x   x   x   x   x
x default     x Pod/nginx                                    x 3 x 16 x 33 x 93 x   x   x   x 4  x 11 x   x   x   x   x   x   x
mqqqqqqqqqqqqqvqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqvqqqvqqqqvqqqqvqqqqvqqqvqqqvqqqvqqqqvqqqqvqqqvqqqvqqqvqqqvqqqvqqqj
Severities: C=CRITICAL H=HIGH M=MEDIUM L=LOW U=UNKNOWN
```