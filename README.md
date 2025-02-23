# FreesiaGuide
Freesia的食用文档

## Velocity
这里需要分两个东西，一个是master_controll_service（后面我们称其为mcs）和msession（后面我们称其为ms）,在mcs中我们用于velocity和worker节点的通信用以控制其行为，ms则是用于转发玩家的ysm数据包到装有ysm模组的worker节点上来进行缓存同步的操作，
这里需要确保velocity可以连接到worker的mc端口上而且切记<b>不要把worker放入到velocity的子服列表中也不要把mcs和ms暴露于公网之下，否则会造成安全问题</b>

详细说明的配置文件如下:
~~~toml
[functions]
        # 踢出未安装ysm模组的玩家
	kick_if_ysm_not_installed = false
	# 握手超时
	ysm_detection_timeout_for_kicking = 30000
[messages]
        # 语言, 目前有zh_CN和en_US
	language = "zh_CN"
[worker]
        # 主节点控制服务的地址
	worker_master_ip = "localhost"
	worker_master_port = 19200
	# worker服务端的地址（分别对应worker的server.properties中的server-port和server-ip）
	# 特别注意: 不要把worker和主控制节点的端口暴露于公网，否则会造成安全性问题
	worker_msession_port = 19199
	worker_msession_ip = "localhost"
~~~

## Worker
Worker是一个安装了ysm和安装了worker模组（Freesia-Worker）的1.21的fabric服务端，用于处理一些ysm的逻辑诸如缓存同步等以及用于实体状态更新的实体数据nbt的生成，在worker上我们禁用了绝大多数的功能以尽可能减少其占用，在这上面只需要处理ysm的数据包和登录阶段的数据包用于模拟类似multipaper的多slave场景

在Worker上不会进行任何大多数游戏逻辑也不会进行大多数保存，但仍会有tickloop，由于ysm逻辑的实现，绝大多数的性能瓶颈也源自于worker这一侧但是由于Freesia的异步逻辑来说并不是很明显

worker上模组的详细配置文件如下:

~~~toml
[worker]
        # 和Freesia-Velocity中的分别对齐
	worker_master_ip = "localhost"
	worker_master_port = 19200
	# 控制节点重连间隔时间，单位是秒
	controller_reconnect_interval = 1
	# 玩家数据缓存的保持时间，单位是秒
	player_data_cache_invalidate_interval_seconds = 30
~~~

worker服务端的配置文件如下:

server.properties:
~~~properties
# 和Freesia-Velocity中的worker_msession_ip对齐
server-ip=127.0.0.1
# 和Freesia-Velocity中的worker_msession_port对齐
server-port=19199
~~~

## 子服
在子服上我们需要处理Velocity侧发来的玩家tracker检查请求以及通知Velocity更新玩家tracker，所以在子服上必须安装Freesia-Backend

这里没有什么好说明的目前

## 其他
如果你仍然不明白，在这个仓库中有一份样例服务端，它就躺在examples中呢^-^
注意： 个别文件由于过大没有上传上(ysm的jar)
