---
title: RTSP与RTMP的区别
date: 2019-05-14 16:08:06
tags: 协议
keywords: RTSP与RTMP
description:
---


#### 前言

>**RTSP （Real-Time Stream Protocol）** 由Real Networks 和 Netscape共同提出的，基于文本的多媒体播放控制协议。RTSP定义流格式，流数据经由RTP传输；RTSP实时效果非常好，适合视频聊天，视频监控等方向。
>**RTMP（Real Time Message Protocol）** 由 Adobe 公司提出，用来解决多媒体数据传输流的多路复用（Multiplexing）和分包（packetizing）的问题，优势在于低延迟，稳定性高，支持所有摄像头格式，浏览器加载 flash插件就可以直接播放。

#### 区别

| 区别  |  RTSP | RTMP  |
| --- | --- | --- | 
| 协议 | 实时流传输协议（Real Time Streaming Protocol） |  路由选择表维护协议（Routing Table Maintenance Protocol） |
| 类型 | 流媒体协议 | 流媒体协议 |
| 公开  | 公开协议，并有专门机构做维护  | Adobe的私有协议,未完全公开 |
| 格式 | ts,mp4 | flv，f4v |
| 通道 |  需要2-3个通道，命令和数据通道分离 | 在TCP一个通道上传输命令和数据 |



#### 参考：
[https://www.jianshu.com/p/70c9a2fd918b](https://www.jianshu.com/p/70c9a2fd918b)
[https://www.cnblogs.com/gongyuhonglou/p/5605320.html](https://www.cnblogs.com/gongyuhonglou/p/5605320.html)




