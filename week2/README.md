# 极客时间运维进阶训练营第二周作业
[Dockerfile](./Dockerfile)

## 基于 dockerfile，实现分层构建的 nginx 业务镜像
`docker build -t my-nginx .`


截取部分 log

```shell
Step 7/13 : RUN groupadd  -g 2088 nginx && useradd  -g nginx -s /usr/sbin/nologin -u 2088 nginx && chown -R nginx.nginx /apps/nginx
 ---> Running in 1989c02ca157
Removing intermediate container 1989c02ca157
 ---> 6a9da0038a0f
Step 8/13 : ADD nginx.conf /apps/nginx/conf/
 ---> 4c0af26339a0
Step 9/13 : ADD frontend.tar.gz /apps/nginx/html/
 ---> 60f1dd85a3b4
Step 10/13 : EXPOSE 80 443
 ---> Running in 9050375e6de0
Removing intermediate container 9050375e6de0
 ---> 38115af922f3
Step 11/13 : COPY docker-entrypoint.sh /docker-entrypoint.sh
 ---> 2d7acba9ac77
Step 12/13 : RUN chmod a+x /docker-entrypoint.sh
 ---> Running in 81b3c73be211
Removing intermediate container 81b3c73be211
 ---> d707e63eace5
Step 13/13 : ENTRYPOINT ["/docker-entrypoint.sh"]
 ---> Running in aa5375fa4e7c
Removing intermediate container aa5375fa4e7c
 ---> 5bcf16746bdf
Successfully built 5bcf16746bdf
Successfully tagged my-nginx:latest
```
`docker run --name my-nginx-1 -d -p 8080:80 my-nginx`

執行以下指令可見在 Dockefile 中設置的 label 跟 env 
`docker inspect my-nginx`


```shell
...
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "443/tcp": {},
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "FLAG1=1",
                "FLAG2=2"
            ],
            "Cmd": null,
            "Image": "sha256:27f568faa9a9302c5c464078505a72db0fc82b967c080c9e88dafb304ad029d0",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "MAINTAINER": "Julian Chu",
                "VERSION": "0.0"
            }
        },
...
```

心得：
- `COPY` 與 `ADD` 類似,  但 `ADD` 支援了 remote url 跟自動解壓 tar, 根據 [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#add-or-copy) 建議使用 `COPY`, 如果需要 remote url 跟 tar, 可以利用 `wget`/`curl`
```shell
// not recommend
ADD https://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all

// recommend
RUN mkdir -p /usr/src/things \
    && curl -SL https://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all

```
- RUN 指令會創建新的 layer, 多個指令需要執行最好合併在同一行 RUN 之中, 避免增加 image size
```shell
// bad
RUN apt-get update 
RUN apt-get install -y bzr cvs git mercurial subversion
RUN rm -rf /var/lib/apt/lists/*


// good
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion \
  && rm -rf /var/lib/apt/lists/*
```
- 還可以使用 multiple-stage 在 builder stage 安裝需要的編譯環境, 再將最後的 manifest copy 到只有安裝 runtime 環境中做成最終 image, 也可以有效降低 image size
```shell
# syntax=docker/dockerfile:1

FROM golang:1.16
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go ./
RUN CGO_ENABLED=0 go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=0 /go/src/github.com/alexellis/href-counter/app ./
CMD ["./app"]
```