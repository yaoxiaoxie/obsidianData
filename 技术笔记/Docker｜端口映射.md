# Docker-端口映射

## Docker端口映射

docker容器在启动的时候，如果不指定端口映射参数，在容器外部是无法通过网络来访问容器内的网络应用和服务的。

亦可使用Dockerfile文件中的EXPOSE指令来配置。

### 端口映射可使用-p、-P来实现：

- -p指定要映射的端口，一个指定端口上只可以绑定一个容器
- -P将容器内部开放的网络端口随机映射到宿主机的一个端口上

### 端口映射支持的格式：

```css
ip:hostport:containerport #指定ip、指定宿主机port、指定容器port
ip::containerport #指定ip、未指定宿主机port（随机）、指定容器port
hostport:containerport #未指定ip、指定宿主机port、指定容器port
```

### 端口的映射有以下五种方法：

### 一、将容器暴露的所有端口，都随机映射到宿主机上。

例如：（不推荐使用）

```undefined
docker run -P -it ubuntu /bin/bash 
```

### 二、将容器指定端口随机映射到宿主机一个端口上。

例如：

```undefined
docker run -P 80 -it ubuntu /bin/bash
```

以上指令会将容器的80端口随机映射到宿主机的一个端口上。

### 三、将容器指定端口指定映射到宿主机的一个端口上。

例如：

```undefined
docker run -p 8000:80 -it ubuntu /bin/bash
```

以上指令会将容器的80端口映射到宿主机的8000端口上。

### 四、将容器ip和端口，随机映射到宿主机上。

```undefined
docker run -P 192.168.0.100::80 -it ubuntu /bin/bash
```

以上指令会将容器的ip192.168.0.100和80端口，随机映射到宿主机的一个端口上。

### 五、将容器ip和端口，指定映射到宿主机上。

```undefined
docker run -p 192.168.0.100:8000:80 -it ubuntu /bin/bash
```

以上指令会将容器的ip192.168.0.100和80端口，映射到宿主机的8000端口。

#### 示例：

```bash
#将nginx的80端口映射到宿主机的800端口上
docker run -d -it -p 800:80 nginx 
```

### 查看映射端口配置

```bash
docker port container_ID #容器ID
#结果输出
80/tcp -> 0.0.0.0:800
```

