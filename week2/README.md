[Dockerfile](./Dockerfile)

`docker build -t my-nginx .`

`docker inspect my-nginx`

labels:
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
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": null,
            "Image": "sha256:d707e63eace5a5d848058edf7917dc20bd7b02724983fba9cca9fbe4173a8110",
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
