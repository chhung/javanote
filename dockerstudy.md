# Docker
## Basic commands
#### Search image in default docker registry
```bash
[docker@localhost ~]$ docker search httpd
```
![](https://i.imgur.com/GwbNTnQ.png)
#### Pull image from docker registry
```bash
[docker@localhost ~]$ docker pull httpd
```
![](https://i.imgur.com/o0FYvnh.png)
#### List images
```bash
[docker@localhost ~]$ docker images
```
![](https://i.imgur.com/kFTh6jy.png)
#### Build a container from image and run it
`-p [docker server port]:[container port]`  
`-d` 背景執行  
`-it` 以互動的模式執行，在 Docker 中執行 bash  
`--name` 指定container名稱  

```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

[docker@localhost ~]$ docker run -dit -p 6666:80 --name testweb httpd:latest
```

#### List container
```shell
[docker@localhost ~]$ docker ps
```

#### List all container 
```shell
[docker@localhost ~]$ docker ps -a
```

### Start/Restart/Stop/Delete container
#### Start
```shell
[docker@localhost ~]$ docker start testweb
```

#### Restart
```shell
[docker@localhost ~]$ docker restart testweb
```

#### Stop
```shell
[docker@localhost ~]$ docker stop testweb
```

#### Del
```shell
[docker@localhost ~]$ docker rm testweb
```

#### Delete image
```shell
[docker@localhost ~]$ docker rmi httpd:latest
```

#### Show container information
```shell
[docker@localhost ~]$ docker inspect testweb
```

#### View container log
```shell
[docker@localhost ~]$ docker logs -f testweb
[docker@localhost ~]$ docker logs testweb
[docker@localhost ~]$ docker logs --tail 2 testweb
[docker@localhost ~]$ docker logs --since 2m testweb
```

#### 顯示container的資源使用狀況
```shell
[docker@localhost ~]$ docker stats
[docker@localhost ~]$ docker stats -a
```

### 拷貝檔案，使用docker command
* 不管container有沒有執行，都可以使用。
* 預設會覆蓋掉檔案，且不會有任何提示，小心使用。

#### 把檔案從container拷貝出來
```shell
[docker@localhost ~]$ docker cp [container]:[file path] [docker server path]
```

#### 把檔案拷貝到container裡面
```shell
[docker@localhost ~]$ docker cp [docker server path] [container]:[file path]
```

#### 使用nsenter進入docker
```shell
[docker@localhost ~]$ docker inspect --format "{{ .State.Pid }}" testweb
16667
[docker@localhost ~]$ sudo nsenter --target 16667 --mount --uts --ipc --net --pid
```

#### 使用exec入docker
```shell
[docker@localhost ~]$ docker exec -it testweb bash
```

#### 執行docker裡面的命令
```shell
[docker@localhost ~]$ docker exec -it testweb sh -c "pwd"
```

#### 設定docker啓動時的環境變數
```shell
[docker@localhost ~]$ docker run -eMYSQL_ROOT_PASSWORD=123456 -d mysql
```

#### 掛載container目錄或檔案到本地端
```shell
[docker@localhost ~]$ docker run -v /var/www:/usr/share/nginx/html --name mynginx nginx
```

---
## 進階操作
### 多個Container專用的網路
> [name=Chia Hsiang Hung]在Docker中可以創建多的網卡。

#### 查詢目前的Docker網卡
```shell
[docker@localhost ~]$ docker network ls
```

假設，目前有三台電腦，其中兩台都可與另一台連線，但是這兩台彼此是無法連線的；應用的場景之一，像是HA功能，有一台做為zookeeper server，另外兩台則是和這一台伺服連線，選出目前要提供服務的是哪一台，若提供服務的電腦斷線了，另一台可以透過zookeeper server通知繼續提供服務。
![](https://i.imgur.com/sWgO08f.png)

由於Container啓動沒有指定加入哪一個網路，預設就是docker本身一個Bridge網路，而且大家都在同一個子網域，所以互相可以通訊。所以，現在我們要創建三個獨立的網路(不同的子網域)，使用以下建立docker網路
```shell
[docker@localhost ~]$ docker network create NIC-1
[docker@localhost ~]$ docker network create NIC-2
[docker@localhost ~]$ docker network create NIC-3
```
這裡就先用busybox image來假裝是我們的應用程式和zookeeper。
```shell
[docker@localhost ~]$ docker run -dit --net=NIC-1 --name myZookeeper busybox
[docker@localhost ~]$ docker run -dit --net=NIC-2 --name service-1 busybox
[docker@localhost ~]$ docker run -dit --net=NIC-3 --name service-2 busybox
[docker@localhost ~]$ docker network connect NIC-1 service-1
[docker@localhost ~]$ docker network connect NIC-1 service-2
```
> 如此一來，service-1 and service-2兩個就看不見對方，因為在不同的網域，但是兩台都可以看到myZookeeper server。

如果一開始忘了加入指定的網域時，可以用以下指令加上去。
```shell
docker network connect [OPTIONS] NETWORK CONTAINER
```

---
## Docker Data Volume
一般屬於docker data volume的資料，會被建立在`/var/lib/docker/volumes`下。  

docker data volume不論是自動產生(Hash Code名稱)，或是手動建立的，都不會因為container停止或刪除而自動刪除。

#### 建立有名稱的Volume
```shell
$ docker volume create httpd-html
$ docker volume ls
DRIVER              VOLUME NAME
local               1fca2036d55dced5b7c392782a11076e4450374dc0a694e9f1a6fdaa58e9ba70
local               007af31328b5f24897d61c45ef6a0c798d65d05a504c65a62cd244019c1e0f46
local               24047ac4818c5535ecdcae7051baa0faaca19ae2827e6c430d522a469cc03246
local               httpd-html
```

#### 掛載Volume
```shell
$ docker run -dit -p 40000:80 -v httpd-html:/usr/local/apache2/htdocs --name WebAPI httpd
```

#### Container掛載外部的Data Volume
```shell
$ docker volume mybusybox
$ docker run -dit -v mybusybox:/bin --name myBusyBox busybox
$ docker run -dit -p 40002:80 --mount source=mybusybox,target=/busybox --name www httpd
```

---
## 自制Docker Image
1. 找一個基底映像檔
2. 啓動container
3. 依照需求，做好調整設定
4. commit container變成image

> Usage: `docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`
> * 使用Commit方式，並不會將掛載的Volume資料一併存放到映像檔中(會有該資料夾，但無原本掛載的內容資料)

```shell
$ docker pull nginx
$ docker run --name nginx-hello -p 8080:80 -d nginx
$ docker cp index.html nginx-hello:/usr/share/nginx/html
$ docker commit nginx-hello nginx-commit
$ docker images
```

---
## Docker Build — Dockerfile
### CMD
The CMD instruction has three forms:
* `CMD ["executable","param1","param2"]` (exec form, this is the preferred form)
* `CMD ["param1","param2"]` (as default parameters to ENTRYPOINT)
* `CMD command param1 param2` (shell form)

### ENTRYPOINT
ENTRYPOINT has two forms:
* `ENTRYPOINT ["executable", "param1", "param2"]` (exec form, preferred)
* `ENTRYPOINT command param1 param2` (shell form)

> [name=Chia Hsiang Hung]executable指的就是，/bin/bash, python, etc.，所以如果要執行預先寫定好的script，則必需加上跑這個檔案的執行命令，否則就視為非shell form。
所以，也可以用shell form來執行linux script file。

```dockerfile=1
FROM alpine:3.10.2

# 有兩種格式，<key> <value> and <key>=<value>
# 這裡的環境變數，dockerfile可以用，container裡也可以用
ENV INV_VERSION="1.13.4-1" BUILDER="Zack" OPENJDK=""

# 使用環境變數，有兩種形式階可用，${<key>} or $<key>
LABEL maintainer="Zack <zack.j.hung@newegg.com>" \
			version=${INV_VERSION} \
			service.product="Inventory and Price" \
			description="Inventory and Price datafeed procssor"

# 每加一個RUN，就會在基底映像層加上一層資料層
# 東西不多還好，多的話個人建議用拷貝的方式，以避免映像檔越來越大(也可以用&&串接多個命令達成)
# 例如以下openjdk也可以先預備好，再拷貝到container，設定PATH
# ex. ADD jdk-8u152-linux-x64.tar.gz /
RUN apk --no-cache add openjdk8 --repository=http://dl-cdn.alpinelinux.org/alpine/edge/community

#ADD openjdk-12.0.2_linux-x64_bin.tar.gz /

# Container啓動後預設要執行的指令，一個dockerfile只能有一個CMD
#CMD

COPY myAPP/ /myAPP/
WORKDIR /myAPP

# Container啓動後預設要執行的指令，一個dockerfile只能有一個ENTRYPOINT
# 
ENTRYPOINT ["/bin/sh", "service.sh"]

# 若這個映像檔是作為其他映像檔的基底時，便需要定義ONBUILD
#ONBUILD

# container定義要對外的port number，也可以設定udp --> 80/udp
# 但這僅是定義，同時需要docker run -p 8080:80才真正有作用
# 因此，我通常不在Dockerfile做port mapping，而是移到docker run才來做
# docker run -P，大小寫則會直接用dockerfile裡的port號定義，隨機對映到外部的port號
EXPOSE 80/tcp
```

```shell
$ docker build -t dok:<tag> . --no-cache
$ docker run -it --rm -e BUILDER=Alice --name test dok
```

> [name=Chia Hsiang Hung]
> * 上面run的指令，當中給了container環境變數，這個會覆蓋掉Dockerfile裡面的BUILDER變數:<tag>如果拿掉，那就是預設 latest。
> * Dockerfile裡的 EN V變數在下docker build時並無法更改，此時可將 ENV 改成用 ARG，然後在`$ docker build —build-arg <key>=<value>`可以指定新的值覆蓋過去。

---
## 匯入匯出image
### 匯出Image
> Usage: `docker save [OPTIONS] IMAGE [IMAGE...]`

```bash
$ docker save -o dok-2.3.5.tar dok:2.3.5
$ ll
total 104220
drwxrwxr-x  3 docker  docker        80 Sep 29 23:13 build-image
drwxrwxr-x  2 docker  docker         6 Sep 28 08:03 busybox
-rw-rw-r--. 1 docker  docker        14 Jul 26 06:08 dd
drwxr-xr-x. 2 docker  docker         6 Jul 21 21:49 Desktop
drwxr-xr-x. 2 docker  docker         6 Jul 21 21:49 Documents
-rw-------  1 docker  docker 106704896 Sep 30 01:25 dok-2.3.5.tar
drwxr-xr-x. 2 docker  docker         6 Jul 21 21:49 Downloads
drwxrwxr-x  4 docker  docker        38 Sep 28 07:47 httpfile
drwxr-xr-x. 2 docker  docker         6 Jul 21 21:49 Music
drwxrwxr-x  7 polkitd input       4096 Sep 10 07:42 mysql-db
drwxr-xr-x. 2 docker  docker         6 Jul 21 21:49 Pictures
drwxr-xr-x. 2 docker  docker         6 Jul 21 21:49 Public
-rw-r--r--. 1 docker  docker      7511 Jul  5 04:56 server.xml
drwxr-xr-x. 2 docker  docker         6 Jul 21 21:49 Templates
drwxr-xr-x. 2 docker  docker         6 Jul 21 21:49 Videos
```

### 匯入Image
> Usage: `docker load [OPTIONS]`
```bash
$ docker load -i dok-2.3.5.tar
49967ba36dd5: Loading layer [==================================================>]  100.8MB/100.8MB
587e2ebca0c4: Loading layer [==================================================>]  3.072kB/3.072kB
Loaded image: dok:2.3.5
```

### 從Container直接匯出成Image
> Usage: `docker export [OPTIONS] CONTAINER`
```bash
$ docker export -o wwwServer.tar wwwServer
```

### 匯入成Image
> Usage: `docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]`
```bash
$ docker import wwwServer.tar httpd:3.5.7
or
$ docker import wwwServer.tar -c 'CMD ["httpd-foreground"]' httpd:3.5.7
```

這樣就能匯入成docker image，致於為什麼要再加上-c參數呢？  
因為，Container啓動命令在用export匯出時會被遺失掉，因此，在下docker run時就會出現
```bash
docker: Error response from damnon: No command specified.
See 'docker run --help'.
```

---
## 推到私有容器管理
把private registry container存放image的目錄mapping到外部，這樣就算registry container刪除，日後也可以保留image
```bash
$ sudo mkdir -p /docker/registry
$ docker pull registry:2
$ docker run -d -p 5000:5000 -v /docker/registry:/var/lib/registry --restart always --name registry registry:2
$ curl -X GET 127.0.0.1:5000/v2/
{}
```

上Tag，再push
```bash
$ docker tag dok:2.3.5 localhost:5000/dok:2.3.5
$ docker push localhost:5000/dok:2.3.5
```
![](https://i.imgur.com/S1pU0vh.png)

從另一台電腦下載private docker registry
```bash
$ docker pull 172.20.10.8:5000/dok:2.3.5
Error response from daemon: Get https://172.20.10.8:5000/v2/: http: server gave HTTP response to HTTPS client
```
有這個錯誤，主要是安全性上的問題，修改Client的docker設定
```bash
# vi /etc/docker/daemon.json
```

daemon.json
```json
{
  "live-restore": true,
  "group": "dockerroot",
  "insecure-registries": ["172.20.10.8:5000"]
}
```

```bash
$ systemctl restart docker
$ docker pull 172.20.10.8:5000/dok:2.3.5
2.3.5: Pulling from dok
9d48c3bd43c5: Pull complete
5acc03bd08ba: Pull complete
0697bf609643: Pull complete
Digest: sha256:fe8081022ffe92bf715158e3b94e3afeb826a4f819cd7b21346bba41cf763890
Status: Downloaded newer image for 172.20.10.8:5000/dok:2.3.5
```

### Check private docker registry image information
https://docs.docker.com/registry/spec/api/#detail
```bash
$ curl -X GET http://172.20.10.8:5000/v2/_catalog
{"repositories":["dok"]}
$ curl -X GET http://172.20.10.8:5000/v2/dok/tags/list
{"name":"dok","tags":["2.3.5"]}
```
