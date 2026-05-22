小米球 Docker 镜像 README.md

# 小米球 Docker 镜像使用文档

官方网站：https://www.xiaomiqiu.cn/
项目地址：https://github.com/MagicToDo/xiaomiqiu-docker.git



[TOC]



轻量、开箱即用的小米球内网穿透工具 Docker 镜像，仅需配置服务器地址与 Token 即可快速运行。
---

## 拉取镜像

```bash
# 拉取最新版本小米球 Docker 镜像
docker pull magictodo/xiaomiqiu:latest
```

---

## 启动方式（二选一）

### 方式 1：Docker Run 启动（简易快捷，适合新手）

```bash
# 后台运行（-d）、开机自启（--restart=always）、固定容器名（--name=xiaomiqiu）
# 配置服务器地址（SERVER_ADDR）和认证 Token（AUTH_TOKEN）
docker run -d \
  --restart=always \
  --name=xiaomiqiu \
  -e SERVER_ADDR=ngrok.xiaomiqiu123.top:5432 \
  -e AUTH_TOKEN=你的token \
  magictodo/xiaomiqiu:latest
```

### 方式 2：Docker Compose 启动（稳定持久，推荐长期使用）

1. 新建 docker-compose.yml 文件，写入以下内容：

```yaml
version: '3'

services:
  xiaomiqiu:
    # 使用小米球 Docker 镜像
    image: magictodo/xiaomiqiu:latest
    # 固定容器名称，后续管理命令统一使用此名称
    container_name: xiaomiqiu
    # 环境变量配置（服务器地址和认证 Token）
    environment:
      - SERVER_ADDR=ngrok.xiaomiqiu123.top:5432
      - AUTH_TOKEN=你的token
    # 容器异常、服务器重启后自动启动，保证服务稳定
    restart: always
```

2. 执行启动命令：

```bash
# 后台启动容器，不占用终端
docker-compose up -d
```

---

## 环境变量说明

| 变量名      | 说明                                | 必填 |
| ----------- | ----------------------------------- | ---- |
| SERVER_ADDR | 小米球服务器地址（格式：域名:端口） | ✅    |
| AUTH_TOKEN  | 小米球官网控制台获取的认证 Token    | ✅    |

---

---


## 容器管理命令

> 说明：无论使用 Docker Run 还是 Docker Compose 启动，以下命令完全通用（容器名统一为 xiaomiqiu）

```bash
# 查看容器实时运行日志（排查启动/运行问题最常用）
docker logs -f xiaomiqiu

# 停止小米球容器
docker stop xiaomiqiu

# 重启小米球容器
docker restart xiaomiqiu

# 删除小米球容器（需重新配置时使用，删除后需重新启动）
docker rm -f xiaomiqiu

# 查看容器运行状态（确认是否正常启动）
docker ps | grep xiaomiqiu
```

---

## 注意事项

1. 请务必将命令/配置文件中的「你的token」替换为小米球官网控制台获取的真实认证 Token，否则无法正常穿透。
2. SERVER_ADDR 为小米球官方提供的服务器地址，请勿随意修改，修改后会导致连接失败。
3. 推荐保留「restart: always」（Docker Compose）或「--restart=always」（Docker Run）配置，确保容器断电、崩溃后自动重启，保障服务稳定。
4. 容器名统一设置为 xiaomiqiu，方便后续执行管理命令，无需额外查找容器ID。（**如果需要修改或添加多个服务请注意处理容器名称避免冲突**）



------



## 附加：Docker 网络说明（bridge）

> [!NOTE]
>
> 在部分docker管理面板，网络也称为接口/网络接口/网络模式

默认情况下，本镜像使用 Docker 的 `bridge` 网络模式运行。

也就是说，如果启动命令中没有额外指定网络模式，例如没有写 `--network host`、`--network macvlan`，Docker 会自动将容器加入默认的 `bridge` 网络。

### bridge 模式特点

```text
宿主机
  └── Docker bridge 网络
        └── xiaomiqiu 容器
```

如果容器一定要指定网络模式，推荐bridge，也可使用host，此处只对bridge进行说明。

### 可选：指定 Docker 虚拟网络

### 什么是虚拟网段？

Docker 容器运行时，通常不会直接使用宿主机的真实局域网 IP，而是由 Docker 创建一个独立的虚拟网络。

例如：

```text
宿主机真实局域网：192.168.9.10
Docker 虚拟网段：172.30.10.0/24
容器 IP：172.30.10.2
Docker 网关：172.30.10.1
```

这里的 `172.30.10.0/24` 就是 Docker 给容器使用的**虚拟网段**。

它只在 Docker 内部使用，主要作用是让容器拥有自己的内部 IP，并通过 Docker 网关访问外部网络。

简单理解：

```
容器 → Docker 虚拟网段 → 宿主机 → 局域网 / 外网
```

默认情况下，小米球容器会使用 Docker 默认 `bridge` 网络，一般无需额外配置。

如果默认 Docker 网段与局域网、VPN、软路由或其他 Docker 网络冲突，可以手动指定一个独立的虚拟网段。

------

### Docker Run 示例

先创建虚拟网络：

```
docker network create \
  --driver bridge \
  --subnet 172.30.10.0/24 \
  --gateway 172.30.10.1 \
  xiaomiqiu-net
```

启动容器时指定网络：

```
docker run -d \
  --restart=always \
  --name=xiaomiqiu \
  --network xiaomiqiu-net \
  -e SERVER_ADDR=ngrok.xiaomiqiu123.top:5432 \
  -e AUTH_TOKEN=你的token \
  magictodo/xiaomiqiu:latest
```

------

### Docker Compose 示例

```
version: '3'

services:
  xiaomiqiu:
    image: magictodo/xiaomiqiu:latest
    container_name: xiaomiqiu
    environment:
      - SERVER_ADDR=ngrok.xiaomiqiu123.top:5432
      - AUTH_TOKEN=你的token
    networks:
      - xiaomiqiu-net
    restart: always

networks:
  xiaomiqiu-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.30.10.0/24
          gateway: 172.30.10.1
```

------

> [!WARNING]
>
> `172.30.10.0/24` 只是示例虚拟网段，实际使用时应避免与局域网、VPN、软路由或其他 Docker 网络重复。
>
> 例如，如果局域网已经使用：`192.168.9.0/24`，则不要把 Docker 虚拟网络也设置为相同网段。
>
> 系统转发网络请求时，会根据**路由表**判断“这个 IP 应该从哪张网卡出去”。
>
> 如果 Docker 虚拟网段和局域网 / VPN / 软路由网段重复，系统就分不清目标 IP 到底是：
>
> ```
> Docker 内部容器地址
> ```
>
> 还是：
>
> ```
> 真实局域网设备地址
> ```
>
> 
>
> 如果只是普通使用小米球容器，通常保持默认 `bridge` 网络即可；只有在默认网络冲突、需要隔离多个容器服务，或者需要明确指定 Docker 网段时，才需要手动创建虚拟网络。
