# media-server

media-server 是一个基于 [brpc](https://github.com/brpc/brpc) 的直播流媒体服务器，应用于百度云的 [直播流服务](https://cloud.baidu.com/product/lss.html) of Baidu Cloud.

## 主要特性

* 支持 [origin server](docs/cn/origin_server.md),可以推送和播放流媒体
* 支持 [edge server](docs/cn/edge_server.md) 用于代理推送/拉取请求
* 支持 [rtmp](https://www.adobe.com/devnet/rtmp.html)/[flv](https://en.wikipedia.org/wiki/Flash_Video)/[hls](https://en.wikipedia.org/wiki/HTTP_Live_Streaming) 播放
* 支持 rtmp 推流
* 流通过 [vhost/app/stream_name](docs/cn/vhost_app_stream.md)唯一确定
* 可配置的推拉流 [retry policy](docs/cn/retry_policy.md)
* 支持 [simplified rtmp protocol](docs/cn/simplified_rtmp.md) 简化了 rtmp 握手过程
* 支持 [visual interface](docs/cn/http_service.md)（通过 HTTP）查看当前服务器/流媒体状态
* 支持 [low latency hls](docs/cn/low_latency_hls.md)（比 rtmp/flv 慢约一秒）
* 支持 [video/audio only](docs/cn/av_only.md) 直播流
* 可配置的 [frame queue buffer](docs/cn/frame_queue.md) 长度（通常为几秒）
* 支持 [keep pulling](docs/cn/keep_pulling.md) streams for several seconds 在没有用户观看时
* 支持将 [stream status](docs/cn/stream_status.md) 输出到日志，用于监控
* 支持不同的 [re-publish policy](docs/cn/republish_policy.md)
* 支持 https
* 支持  [brpc](https://github.com/brpc/brpc) 带来的所有功能

## 快速开始

支持的操作系统：Linux，MacOSX。

* 安装 [brpc](https://github.com/brpc/brpc/blob/master/docs/cn/getting_started.md) ，这是 media-server 的主要依赖项。
* 使用 cmake 编译 media-server:
```shell
mkdir build && cd build && cmake .. && make -sj4
```
* 以源服务器模式运行 media-server，使用最小配置（默认端口为 8079）:
```shell
./output/bin/media_server
```
然后，你可以推送/播放流。

### 主要选项

请运行
```
./output/bin/media_server --help
```
以获取详细的配置说明。

* -proxy_to
当未指定或为空时，media-server 运行在源服务器模式，聚合推流（如 OBS、ffmpeg）并接受播放请求（如 cyberplayer、ffplay）。

* -proxy_lb
当 -proxy_to 是一个命名服务（例如 http://...）时，需要指定负载均衡算法。可选的算法有 rr、random、la、c_murmurhash 和 c_md5。详情请参见[客户端负载均衡](https://github.com/brpc/brpc/blob/master/docs/en/client.md#user-content-load-balancer) for details.

* -port
指定 media-server 的服务端口。brpc 的特点是支持在同一端口上使用所有协议，因此该端口也可用于通过 HTTP 访问内置服务。只有在 8000-9000 范围内的端口才能通过浏览器访问，这意味着如果服务端口是外部端口，则需要配置 -internal_port 以防止内置服务泄露详细的服务信息

* -internal_port
此端口可以配置为只能在内部网络上访问的端口。在这种情况下，-port 端口不再提供内置服务，仅通过此端口访问。

* -retry_interval_ms
当 media-server 以边缘模式运行时，推拉请求到上游服务出现错误时，将会重试，直到客户端不再需要此流。此选项指定连续重试的最小时间间隔，默认为 1 秒。

* -share_play_connection
设置为 true 时，多个流连接到同一服务器时将复用相同的 rtmp 播放连接。

* -share_publish_connection
设置为 true 时，多个流连接到同一服务器时将复用相同的 rtmp 推流连接。

* -timeout_ms
创建流的超时时间，当 media-server 以边缘模式运行时。默认值为 1000ms。

* -server_idle_timeout
没有数据传输的连接将在此超时后关闭，默认值为 -1（关闭此功能）。

* -cdn_merge_to
当配置此选项时，media-server 启动两个端口，一个用于外部服务请求，另一个用于聚合请求。通常，聚合服务器通过一致性哈希方式找到，用于缓存服务中广泛使用。此选项常用于 CDN 节点。

* -cdn_merge_lb
负载均衡算法，详见 -proxy_lb 选项的说明。

* -flagfile
media-server 使用 gflags 选项，可以在命令行中指定，也可以在在线部署时使用 -flagfile 配置文件格式。

## Examples

* 以 [origin server](docs/cn/origin_server.md)  和 [edge server](docs/cn/edge_server.md)模式运行 media-server。

## 其他文档

* 工具
    * [puller](docs/cn/puller.md)
    * [pusher](docs/cn/pusher.md)
    * [random_test](docs/cn/random_test.md)
    * [rtmp_press](docs/cn/rtmp_press.md)
