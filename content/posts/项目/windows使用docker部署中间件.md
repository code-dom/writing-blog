---

title: "Windows使用docker部署中间件"
subtitle: ""
date: 2025-12-21T22:03:51+08:00
draft: false

tags: []
categories: [项目开发]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

license: '<a rel="license external nofollow noopener noreffer" href="https://creativecommons.org/licenses/by-nc/4.0/" target="_blank">CC BY-NC 4.0</a>'
---

------

# 背景

**受限于现有设备的硬件性能**（一台 M1/8G Mac 和一台 i3-9100f/8G Windows），在开发应用时我遇到了明显的瓶颈。尤其是当需要同时运行 MySQL、Redis、Kafka、Elasticsearch 等多个中间件，再启动 Web 应用时，**单台设备的 CPU 和内存资源均难以独立支撑**。

因此，我计划将 Windows 台式机作为**专用的中间件服务器**，通过 Docker 部署所有依赖服务，而 Mac 仅专注于代码编写与应用运行，以此分担负载，实现流畅开发。

# 总览

{{< style "color: red; font-weight: bold;" "span" >}}client可以ping通server -> client正常访问server的某些端口 -> server正常部署、本机访问正常 -> client访问server中间件{{< /style >}}

# 保证连通性

## 内网穿透

选择一个内网穿透工具，我使用的Tailscale，官方服务器帮助打洞，无需涉及网络底层，只需要在需要互相访问的电脑上登陆Tailscale即可获得一个互相访问的IP。

**使用`ping IP`检测是否联通，如果出现问题，应是内网穿透工具的问题，请先解决该问题再继续。**

## 防火墙（windows版）

### 开启防火墙

通过内网穿透工具我们可以从其他设备访问到自己的服务器，本次使用的是windows作为服务器。

按下面的图示从控制面板找到防火墙设置，然后新增**入站规则**。

<img src="https://cdn.jsdelivr.net/gh/code-dom/picGo/img/202512221026275.png" alt="图1" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/code-dom/picGo/img/202512221029228.png" alt="图2" style="zoom:50%;" />

### 测试防火墙

写一个简单的go测试脚本

```go
package main

import (
	"fmt"
	"net"
)

func main() {
	ln, err := net.Listen("tcp", "0.0.0.0:3303")
	if err != nil {
		panic(err)
	}
	fmt.Println("Listening on 0.0.0.0:3303")
	for {
		conn, err := ln.Accept()
		if err != nil {
			continue
			fmt.Println("[Error]", err)
		}
		fmt.Println("[Success] Connected from %s\n", conn.RemoteAddr())
		conn.Write([]byte("hello from mac\n"))
		conn.Close()
	}
}

```



#### **Ps**

1. 通过测试应用测试时，**如果弹出允许程序接受传入网络连接选了接受**，那么不用开启防火墙也能访问到该程序。反之，如果拒绝了就需要防火墙有其他准入的规则。**为了测试防火墙规则是否正常，选择拒绝**。

# windows使用docker部署中间件

我使用docker-compose部署多个中间件。经过上面的联通，只要docker部署正常就可以访问到（但是windows docker似乎有点bug，我一开始连通正常但一直访问不到mysql等的端口，后面重启后过了一天就好了）。

ps：

1. `docker-compose up -d --force-recreate`重启并使修改后的配置生效。
2. windows的docker有个bug，使用`netstat -an | findStr "3306"`找不到docker接受的端口。

## 测试+遇到的bug

### mysql

直接使用`mysql -h 100.82.159.69 -P 3309 -uroot -p`进行连接测试。

ps：

1. **mysql可能有些奇怪的问题，可以删除挂载卷对应的本机文件进行完全重制。**

#### ERROR 1130

`ERROR 1130 (HY000): Host '172.18.0.1' is not allowed to connect to this MySQL server`

这个是因为我在docker里使用mysql_native_password认证，但是mac中的mysql是9点几，不支持这个插件。只需要在docker里修改使用更新的，例如mysql8默认的caching_sha2_password。

### Kafka

我是直接启动web应用测试的。

#### advertised.listeners里不能出现0.0.0.0

如果出现，客户端拿到这个“不可路由”地址后就不知道该连那一台机器。

`KAFKA_ADVERTISED_LISTENERS: PLAINTEXT_INTERNAL://video_kafka:9092,PLAINTEXT_EXTERNAL://100.82.159.69:9094`

PLAINTEXT_EXTERNAL不能为0.0.0.0，我直接改成Windows 的IP（内网穿透的），然后就可以正常使用。

因为KAFKA_ADVERTISED_LISTENERS就是告诉客户端去哪找本机的kafka

#### docker-compose.yaml

{{< highlight "yaml">}}
video_kafka:

    # 使用 Apache 官方镜像，3.9.0 版本默认使用 KRaft 模式（不再依赖 Zookeeper）
​    image: apache/kafka:3.9.0
    # 指定容器的主机名和容器名，方便 Docker 内部的其他容器（如 kafka-ui）通过名字访问它
​    hostname: video_kafka
​    container_name: video_kafka

    ports:
      # 【外部访问端口】
      # 将宿主机(Windows)的 9094 映射到容器的 9094
      # 这是你的 Mac 通过 Tailscale IP 访问 Kafka 的入口
      - "9094:9094"
      
    environment:
      # 【基础集群配置】
      # 节点 ID，集群中每个节点必须唯一，单机版设为 1 即可
      KAFKA_NODE_ID: 1
      # 角色配置：同时作为 broker (存数据) 和 controller (管集群)，因为是单机 KRaft 模式
      KAFKA_PROCESS_ROLES: broker,controller
      # 定义参与投票的节点列表，格式为 id@host:port
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@video_kafka:9093
      
      # 【监听器名称定义】
      # 指定哪一个监听器是用作控制器内部通信的
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      # 指定哪一个监听器是用作 Broker 之间内部通信的
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT_INTERNAL
      
      # 【安全协议映射】
      # 定义三个通道的协议，开发环境全部使用 PLAINTEXT (明文不加密)
      # 1. CONTROLLER: 管理通道
      # 2. PLAINTEXT_EXTERNAL: 外部连接通道 (Mac 用)
      # 3. PLAINTEXT_INTERNAL: 内部连接通道 (Docker 内部用)
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT_EXTERNAL:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      
      # 【关键配置：监听地址 (Listeners)】
      # Kafka 进程在容器内部实际监听的网卡和端口
      # 1. 9092: 给 Docker 内部用的（绑定 0.0.0.0 代表监听所有网卡）
      # 2. 9093: 给 Controller 管理用的
      # 3. 9094: 给外部用的（建议绑定 0.0.0.0，确保能接收来自 Windows 端口映射的流量）
      KAFKA_LISTENERS: PLAINTEXT_INTERNAL://0.0.0.0:9092,CONTROLLER://video_kafka:9093,PLAINTEXT_EXTERNAL://0.0.0.0:9094
      
      # 【关键配置：广告地址 (Advertised Listeners)】
      # Kafka 告诉客户端“你应该访问哪个地址来找我”
      # 1. 内部客户端（如 kafka-ui）: 告诉它访问 video_kafka:9092
      # 2. 外部客户端（你的 Mac）: 告诉它访问 Windows 的 Tailscale IP + 9094
      # ★★★ 请确保 100.82.159.69 是你 Windows 的 Tailscale IP ★★★
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT_INTERNAL://video_kafka:9092,PLAINTEXT_EXTERNAL://100.82.159.69:9094
      
      # 【单机版防报错优化】
      # 生产环境通常为 3，单机环境必须强行设为 1，否则会因为节点数不够而一直报错或无法启动
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1         # 消费位移主题副本数
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1 # 事务日志副本数
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1            # 最小同步副本数
      
      # 【其他优化】
      # 消费者组初始化延迟，设为 0 加快开发环境启动速度
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      # 默认创建 Topic 时的分区数
      KAFKA_NUM_PARTITIONS: 3
      # 数据存储路径
      KAFKA_LOG_DIRS: /var/lib/kafka/data
      
    volumes:
      # 数据持久化挂载：防止容器删除后数据丢失
      - ./deploy/data/kafka/data:/var/lib/kafka/data
{{</highlight>}}
