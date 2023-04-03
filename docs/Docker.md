# Docker

>  docker安装在windows上

## 常用命令

> 以安装zookeeper为例

### 拉取镜像

```bash
docker pull zookeeper
```

这将默认拉取最新版本的镜像。

```bash
docker pull zookeeper:3.6.3
```

可以指定拉取的镜像版本。

### 启动容器

1. 启动临时容器

   ```bash
   docker run --rm zookeeper
   ```

   临时容器在容器关闭时，将自动删除。

2. 启动容器

   ```bash
   docker run -d --name myzookeeper -p 2181:2181 -v C:\tmp\config:/config zookeeper
   ```

   启动持久话容器，将容器2181端口映射到主机2181端口。`-v`可以将本地文件挂载到容器内，用`：`分开。

### 进入容器

```bash
docker exec myzookeeper cp /conf/zoo_sample.cfg /config/zoo.cfg
```

进入容器，并将zoo_sample.cfg复制到/config内。