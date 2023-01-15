# マニュアル

https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

# 学習ログ
## Imageサイズ

以下のDockerfileをBuildする

```Dockerfile
FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN CGO_ENABLED=0 go build app.go
CMD ["./app"]
```

```app.go
package main

import (
    "fmt"
    "time"
    "os/user"
)

func main () {
    user, err := user.Current()
    if err != nil {
        panic(err)
    }

    for {
        fmt.Println("user: " + user.Username + " id: " + user.Uid)
        time.Sleep(1 * time.Second)
    }
}
```

```
$ sudo docker build -t app .
Sending build context to Docker daemon  3.072kB
Step 1/6 : FROM ubuntu
latest: Pulling from library/ubuntu
6e3729cf69e0: Pull complete
Digest: sha256:27cb6e6ccef575a4698b66f5de06c7ecd61589132d5a91d098f7f3f9285415a9
Status: Downloaded newer image for ubuntu:latest
 ---> 6b7dfa7e8fdb
Step 2/6 : ARG DEBIAN_FRONTEND=noninteractive
 ---> Running in e110516e080e
Removing intermediate container e110516e080e
 ---> 75980fb90796
Step 3/6 : RUN apt-get update && apt-get install -y golang-go
 ---> Running in a7572bf25626
Get:1 http://archive.ubuntu.com/ubuntu jammy InRelease [270 kB]
Get:2 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]

・・・

Processing triggers for libc-bin (2.35-0ubuntu3.1) ...
Removing intermediate container a7572bf25626
 ---> 7d5bbe40a3bd
Step 4/6 : COPY app.go .
 ---> 4f025df9377b
Step 5/6 : RUN CGO_ENABLED=0 go build app.go
 ---> Running in 217ee6c35359
Removing intermediate container 217ee6c35359
 ---> 3b8eea174cfd
Step 6/6 : CMD ["./app"]
 ---> Running in 69ea40b9aadd
Removing intermediate container 69ea40b9aadd
 ---> 91b0ece3f0e9
Successfully built 91b0ece3f0e9
Successfully tagged app:latest
```

```
$ sudo docker run app
user: root id: 0
user: root id: 0
user: root id: 0
user: root id: 0
user: root id: 0
・・・
```

imageのサイズを確認する

```
$ sudo docker image list |grep app
app          latest    91b0ece3f0e9   2 minutes ago   861MB
```

Dockerfileを以下に書き換えて、再度Buildする。

```Dockerfile
FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN CGO_ENABLED=0 go build app.go

FROM alpine
COPY --from=0 /app .
CMD ["./app"]
```

```
$ sudo docker build -t app .
Sending build context to Docker daemon  3.072kB
Step 1/8 : FROM ubuntu
 ---> 6b7dfa7e8fdb
Step 2/8 : ARG DEBIAN_FRONTEND=noninteractive
 ---> Using cache
 ---> 75980fb90796
Step 3/8 : RUN apt-get update && apt-get install -y golang-go
 ---> Using cache
 ---> 7d5bbe40a3bd
Step 4/8 : COPY app.go .
 ---> Using cache
 ---> 4f025df9377b
Step 5/8 : RUN CGO_ENABLED=0 go build app.go
 ---> Using cache
 ---> 3b8eea174cfd
Step 6/8 : FROM alpine
latest: Pulling from library/alpine
8921db27df28: Pull complete
Digest: sha256:f271e74b17ced29b915d351685fd4644785c6d1559dd1f2d4189a5e851ef753a
Status: Downloaded newer image for alpine:latest
 ---> 042a816809aa
Step 7/8 : COPY --from=0 /app .
 ---> b1e324f0b52f
Step 8/8 : CMD ["./app"]
 ---> Running in 1f13a220bcdc
Removing intermediate container 1f13a220bcdc
 ---> fd45abd410f4
Successfully built fd45abd410f4
Successfully tagged app:latest
```

```
$ sudo docker run app
user: root id: 0
user: root id: 0
user: root id: 0
```

```
$ sudo docker image list |grep app
app          latest    fd45abd410f4   About a minute ago   8.91MB
```

マルチステージビルドでアプリの動きは一緒だけど、Imageサイズを小さくできる。

## Secure Image
Dockerfileを以下のように書き換える。

- バージョンを指定
    - latestだとセキュリティリスクがあるかもしれないので、安定したバージョンを指定
- Userを作成してroot以外で実行する


```Dockerfile
FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN CGO_ENABLED=0 go build app.go

FROM alpine:3.17.1
RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
COPY --from=0 /app /home/appuser/
USER appuser
CMD ["/home/appuser/app"]
```

```
$ sudo docker build -t app .
Sending build context to Docker daemon  3.072kB
Step 1/10 : FROM ubuntu
 ---> 6b7dfa7e8fdb
Step 2/10 : ARG DEBIAN_FRONTEND=noninteractive
 ---> Using cache
 ---> 75980fb90796
Step 3/10 : RUN apt-get update && apt-get install -y golang-go
 ---> Using cache
 ---> 7d5bbe40a3bd
Step 4/10 : COPY app.go .
 ---> Using cache
 ---> 4f025df9377b
Step 5/10 : RUN CGO_ENABLED=0 go build app.go
 ---> Using cache
 ---> 3b8eea174cfd
Step 6/10 : FROM alpine:3.17.1
3.17.1: Pulling from library/alpine
Digest: sha256:f271e74b17ced29b915d351685fd4644785c6d1559dd1f2d4189a5e851ef753a
Status: Downloaded newer image for alpine:3.17.1
 ---> 042a816809aa
Step 7/10 : RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
 ---> Running in 5d1c2c86bda2
Removing intermediate container 5d1c2c86bda2
 ---> 69605840a2d5
Step 8/10 : COPY --from=0 /app /home/appuser/
 ---> 0dda3bd9192c
Step 9/10 : USER appuser
 ---> Running in 6cfb476434f9
Removing intermediate container 6cfb476434f9
 ---> 4b7bdff7bdd1
Step 10/10 : CMD ["/home/appuser/app"]
 ---> Running in a380744d191c
Removing intermediate container a380744d191c
 ---> 423beb33523f
Successfully built 423beb33523f
Successfully tagged app:latest
```

```
$ sudo docker run app
user: appuser id: 100
user: appuser id: 100
user: appuser id: 100
user: appuser id: 100
user: appuser id: 100
```

さらに、ファイルシステムをReadOnlyにする。
一行追加して、再Build

```Dockerfile
FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN CGO_ENABLED=0 go build app.go

FROM alpine:3.17.1
RUN chmod a-w /etc
RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
COPY --from=0 /app /home/appuser/
USER appuser
CMD ["/home/appuser/app"]
```

```
$ sudo docker build -t app .
Sending build context to Docker daemon  3.072kB
Step 1/11 : FROM ubuntu
 ---> 6b7dfa7e8fdb
Step 2/11 : ARG DEBIAN_FRONTEND=noninteractive
 ---> Using cache
 ---> 75980fb90796
Step 3/11 : RUN apt-get update && apt-get install -y golang-go
 ---> Using cache
 ---> 7d5bbe40a3bd
Step 4/11 : COPY app.go .
 ---> Using cache
 ---> 4f025df9377b
Step 5/11 : RUN CGO_ENABLED=0 go build app.go
 ---> Using cache
 ---> 3b8eea174cfd
Step 6/11 : FROM alpine:3.17.1
 ---> 042a816809aa
Step 7/11 : RUN chmod a-w /etc
 ---> Running in 35b2a73e0f95
Removing intermediate container 35b2a73e0f95
 ---> 4c648dc5b874
Step 8/11 : RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
 ---> Running in 0672ed1988ba
Removing intermediate container 0672ed1988ba
 ---> 7d2e7a570114
Step 9/11 : COPY --from=0 /app /home/appuser/
 ---> f57a4c87b31f
Step 10/11 : USER appuser
 ---> Running in 997055524b6f
Removing intermediate container 997055524b6f
 ---> 64e43351c71d
Step 11/11 : CMD ["/home/appuser/app"]
 ---> Running in d60e6204b7c2
Removing intermediate container d60e6204b7c2
 ---> af15073c1ade
Successfully built af15073c1ade
Successfully tagged app:latest
```

バックグランドで実行し、ログインする。
→ReadOnlyになっている。

```
$ sudo docker run -d app
eba81c1dcd58e302c7b89abef34b8811666fbb515c62efaa6c6b389c9dc9e3a8
$ sudo docker exec -it eba81c1dcd58e302c7b89abef34b8811666fbb515c62efaa6c6b389c9dc9e3a8 sh
/ $
/ $ ls -lh |grep etc
dr-xr-xr-x    1 root     root          66 Jan 15 12:46 etc
```

更に、Shell Accessをできないようにする。
一行追加して、再Build

```Dockerfile
FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN CGO_ENABLED=0 go build app.go

FROM alpine:3.17.1
RUN chmod a-w /etc
RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
RUN rm -rf /bin/*
COPY --from=0 /app /home/appuser/
USER appuser
CMD ["/home/appuser/app"]
```

```
$ sudo docker build -t app .
Sending build context to Docker daemon  3.072kB
Step 1/12 : FROM ubuntu
 ---> 6b7dfa7e8fdb
Step 2/12 : ARG DEBIAN_FRONTEND=noninteractive
 ---> Using cache
 ---> 75980fb90796
Step 3/12 : RUN apt-get update && apt-get install -y golang-go
 ---> Using cache
 ---> 7d5bbe40a3bd
Step 4/12 : COPY app.go .
 ---> Using cache
 ---> 4f025df9377b
Step 5/12 : RUN CGO_ENABLED=0 go build app.go
 ---> Using cache
 ---> 3b8eea174cfd
Step 6/12 : FROM alpine:3.17.1
 ---> 042a816809aa
Step 7/12 : RUN chmod a-w /etc
 ---> Using cache
 ---> 4c648dc5b874
Step 8/12 : RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
 ---> Using cache
 ---> 7d2e7a570114
Step 9/12 : RUN rm -rf /bin/*
 ---> Running in 9109f9a0a025
Removing intermediate container 9109f9a0a025
 ---> 8f92bab900c4
Step 10/12 : COPY --from=0 /app /home/appuser/
 ---> 0821d997f1dd
Step 11/12 : USER appuser
 ---> Running in 26e2ddcb1570
Removing intermediate container 26e2ddcb1570
 ---> 242d3c145f16
Step 12/12 : CMD ["/home/appuser/app"]
 ---> Running in b58e5337d471
Removing intermediate container b58e5337d471
 ---> a002aa2b8348
Successfully built a002aa2b8348
Successfully tagged app:latest
```

```
$ sudo docker run -d app
2072a4419872bdbc8f22f639b8290d7cbad2827c244b71f6a80c7a7211e7674b
$ sudo docker exec -it 2072a4419872bdbc8f22f639b8290d7cbad2827c244b71f6a80c7a7211e7674b sh
OCI runtime exec failed: exec failed: unable to start container process: exec: "sh": executable file not found in $PATH: unknown
```

Shellでログインできなくなっている。

アプリは実行できる。

```
$ sudo docker run app
user: appuser id: 100
user: appuser id: 100
user: appuser id: 100
```