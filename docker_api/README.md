# Docker API
# Docker Remote API
Docker Remote APIをTLSクライアント認証を介さずにアクセスできるようにすると危険である。

dockerをREST APIでアクセスできるようにする。
```shell
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

コンテナを１つ立ち上げておく。
```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
1d282ec99d03        nginx               "/docker-entrypoint.…"   9 seconds ago       Up 8 seconds        80/tcp              happy_mestorf
```

バージョン情報取得。
```
$ curl http://localhost:2376/version
{"Platform":{"Name":"Docker Engine - Community"},"Components":[{"Name":"Engine","Version":"19.03.12","Details":{"ApiVersion":"1.40","Arch":"amd64","BuildTime":"2020-06-22T15:44:20.000000000+00:00","Experimental":"false","GitCommit":"48a66213fe","GoVersion":"go1.13.10","KernelVersion":"4.4.0-1101-aws","MinAPIVersion":"1.12","Os":"linux"}},{"Name":"containerd","Version":"1.2.13","Details":{"GitCommit":"7ad184331fa3e55e52b890ea95e65ba581ae3429"}},{"Name":"runc","Version":"1.0.0-rc10","Details":{"GitCommit":"dc9208a3303feef5b3839f4323d9beb36df0a9dd"}},{"Name":"docker-init","Version":"0.18.0","Details":{"GitCommit":"fec3683"}}],"Version":"19.03.12","ApiVersion":"1.40","MinAPIVersion":"1.12","GitCommit":"48a66213fe","GoVersion":"go1.13.10","Os":"linux","Arch":"amd64","KernelVersion":"4.4.0-1101-aws","BuildTime":"2020-06-22T15:44:20.000000000+00:00"}
```

起動しているコンテナ情報取得。
```
$ curl http://localhost:2376/containers/json
[{"Id":"1d282ec99d03d2beedc2d77478a4b46d5df8378dc3bfac23da5bf13f17d27480","Names":["/happy_mestorf"],"Image":"nginx","ImageID":"sha256:08393e824c32d456ff69aec72c64d1ab63fecdad060ab0e8d3d42640fc3d64c5","Command":"/docker-entrypoint.sh nginx -g 'daemon off;'","Created":1596957317,"Ports":[{"PrivatePort":80,"Type":"tcp"}],"Labels":{"maintainer":"NGINX Docker Maintainers <docker-maint@nginx.com>"},"State":"running","Status":"Up About a minute","HostConfig":{"NetworkMode":"default"},"NetworkSettings":{"Networks":{"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"NetworkID":"9ae3505eeac3a2a381e7f5c53aa9f3dbdc4928ea5788dea65b48803e27ac855e","EndpointID":"391362dc0292840699071c58deb68800f46d77c8f66c29b13ed25eb196bc8c0b","Gateway":"172.17.0.1","IPAddress":"172.17.0.2","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"MacAddress":"02:42:ac:11:00:02","DriverOpts":null}}},"Mounts":[]}]
```