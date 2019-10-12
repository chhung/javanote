# Zookeeper
### Pull Docker
```bash
$ docker pull zookeeper
$ docker run --name some-zookeeper --restart always -p 2181:2181 -d zookeeper
```

### Client connection (linux/windows)
```bash
$ bin/zkCli.sh -server 172.20.10.8:2181
```
```bash
D:\apache-zookeeper-3.5.5\bin>"zkCli.cmd" -server 172.20.10.8:2181
```