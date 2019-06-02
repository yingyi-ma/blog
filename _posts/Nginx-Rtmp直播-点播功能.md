---
title: Nginx-Rtmp直播/点播功能
date: 2019-05-17 16:56:51
tags: [直播,服务器]
keywords:
description:
---
<script type="text/javascript" src="/js/src/bai.js"></script>


### 应用环境
系统: Centos 7.2
[Nginx](http://nginx.org/en/download.html)（本案例使用的事是nginx-1.16.0）
[nginx-rtmp-module](https://github.com/arut/nginx-rtmp-module)
[ffmpeg](http://www.ffmpeg.org/download.html) (推流测试使用，或进行流格式转换)
 

### 安装nginx-rtmp-module插件
 
```
 cd nginx-1.16.0
 ./configure --with-http_ssl_module --add-module=../nginx-rtmp-module-master
 make
 sudo make install
```

### 报错处理（供参考）

**报错 ./configure: error: C compiler cc is not found**

```
yum -y install gcc gcc-c++ autoconf automake make
```

**报错 make: *** No rule to make target `build', needed by `default'. Stop.**

编译参数
```
./configure \
--prefix=/usr/local/nginx \
--pid-path=/usr/local/nginx/run \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_flv_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--with-pcre \
--with-http_image_filter_module \
--with-debug \
;
```


**报错 ./configure: error: the HTTP image filter module requires the GD library.
You can either do not enable the module or install the libraries.**

```
yum -y install gd-devel
```

**报错：nginx: [emerg] getpwnam("nginx") failed**
解决：
```
 useradd -s /sbin/nologin -M nginx
 id nginx
```

### RTMP插件介绍

官方配置Demo说明

```
rtmp {

    server {

        listen 1935;

        chunk_size 4000;

        # TV 模式: 一个推送者, 多个订阅者
        application mytv {

            # 打开直播流
            live on;

            # 记录达到1k时进行切分
            record all;
            record_path /tmp/av;
            record_max_size 1K;

            # flv添加当前时间戳
            record_unique on;

            # 只允许一个推送地址
            allow publish 127.0.0.1;
            deny publish all;

            # 允许所有播放
        }

        # 转码（需要ffmpeg第三方工具）
        application big {
            live on;

            #  推流时执行ffmpeg命令 
            #  替换占位符: $app/${app}, $name/${name} 为应用（application），流名称.
            #  ffmpeg接收流降低分辨率到32x32.  
            #  ‘application small’  在下面的配置可以看到
            exec ffmpeg -re -i rtmp://localhost:1935/$app/$name -vcodec flv -acodec copy -s 32x32
                        -f flv rtmp://localhost:1935/small/${name};
                        
            # ffmpeg 能做很多转码，调整大小, 改变容器/解码器 参数等等
            # 可以指定多个执行行
        }

        application small {
            live on;
            # 分辨率降低的视频来自ffmpeg
        }

        application webcam {
            live on;

            # 来自webcam的本地流
            exec_static ffmpeg -f video4linux2 -i /dev/video0 -c:v libx264 -an
                               -f flv rtmp://localhost:1935/webcam/mystream;
        }

        application mypush {
            live on;

            # 每个推流都会自动推送下面两个服务器
            push rtmp1.example.com;
            push rtmp2.example.com:1934;
        }

        application mypull {
            live on;

            # 从远程的服务器拉流在本地播放
            pull rtmp://rtmp3.example.com pageUrl=www.example.com/index.html;
        }

        application mystaticpull {
            live on;

            # 从nginx静态拉流
            pull rtmp://rtmp4.example.com pageUrl=www.example.com/index.html name=mystream static;
        }

        # 视频点播
        application vod {
            play /var/flvs;
        }

        application vod2 {
            play /var/mp4s;
        }

        # 多个推送，多个订阅
        # 没有校验，没有录像
        application videochat {

            live on;

            #  根据通知接收所有的session变量以及特定的带post参数的http请求
            
            #  使用http请求和http返回码来决定是否推送
            on_publish http://localhost:8080/publish;

            # 播放
            on_play http://localhost:8080/play;

            # 推送或播放结束（重复或者断开连接）
            on_done http://localhost:8080/done;

            # 收到上述所有通知标准 connect（）参数以包括播放/发布。
            # 如果发送任何参数 用get-style语法 包括播放和推送
            # 例：
            # rtmp://localhost/myapp/mystream?a=b&c=d

            # 每2分钟录制10个视频关键帧（无音频）
            record keyframes;
            record_path /tmp/vc;
            record_max_frames 10;
            record_interval 2m;

            # 异步通知：flv录制
            on_record_done http://localhost:8080/record_done;

        }


        # HLS

        # 使用HLS需要为分片视频创建临时文件夹(/tmp/hls)，
        # 文件夹的内容 通过HTTP配置块配置（请看http{}部分 ）
        #
        # 进来的流必须用H264/AAC，对于IPhones用的是H264 
        # 请查看ffmpeg示例
        # 此示例从电影中创建RTMP流以供HLS使用
        #
        # ffmpeg -loglevel verbose -re -i movie.avi  -vcodec libx264
        #    -vprofile baseline -acodec libmp3lame -ar 44100 -ac 1
        #    -f flv rtmp://localhost:1935/hls/movie
        #
        # 如果转码需要使用'exec'
        #
        application hls {
            live on;
            hls on;
            hls_path /tmp/hls;
        }

        # MPEG-DASH 类似 HLS

        application dash {
            live on;
            dash on;
            dash_path /tmp/dash;
        }
    }
}

# HTTP可用于访问RTMP的统计数据
http {

    server {

        listen      8080;

        #  这个URL提供RTMP统计资料在XML中
        location /stat {
            rtmp_stat all;

            # 在网页中显示表格样式
            rtmp_stat_stylesheet stat.xsl;
        }

        location /stat.xsl {
            # XML表格样式据去展示RTMP统计数据
            # 将stat.xsl复制到任何需要的位置
            # 在这里设置全目录
            root /path/to/stat.xsl/;
        }

        location /hls {
            # 用于 HLS 片段
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            root /tmp;
            add_header Cache-Control no-cache;
        }

        location /dash {
            # 用于 DASH 片段
            root /tmp;
            add_header Cache-Control no-cache;
        }
    }
}
```



#### RTMP配置
        
```
 rtmp {  
    server {  
        listen 1935;  #监听的端口
        chunk_size 4000;  
        
        ### 直播，不录播：rtmp://192.168.3.20:1935/live/直播名称
        application live {  #rtmp推流请求路径
            # 开启直播
            live on; 
            # 关闭录像
            record off ;
        }

        ### 点播：rtmp://192.168.3.20:1935/video/播放文件名称
         application video {
            # 播放资源路径
            play /development/vedio;
        }
        ### 点播：rtmp://192.168.3.20:1935/record/播放文件名称
        application record {
            # 播放资源路径
            play /development/records;
        }  
        ### 直播+录播：rtmp://192.168.3.20:1935/liverecord/直播名称
        application liverecord {
           # 打开直播流
           live on;
           recorder rec1{
               # 接收视频音频等配置，详情请看官网
               record all;
               # 录像保存地址，注意文件夹权限，否则不能生成文件！
               record_path /development/records;
               # 切分最大值，不设置不切分
               record_max_size 10M;
               # 保证唯一生成带时间戳的文件
               record_unique on;
               # 时间戳后缀
               record_suffix -%Y%m%d_%H%M%S.flv;  
           }
        } 
    }
 }  


```

#### HLS配置

rtmp > server 配置
```
application hls { #rtmp推流请求路径 rtmp://192.168.3.20:1935/hls/
            live on;
            hls on; 
            hls_path /home/hls/test; #视频流文件目录(自己创建)
            # hls_fragment 3s; 
        }
```

http配置
```
http {
     server {
            listen      8080;
            location /hls {
                # 用于 HLS 片段
                types {
                    application/vnd.apple.mpegurl m3u8;
                    video/mp2t ts;
                }
                root /tmp;
                add_header Cache-Control no-cache;
            }
     }
 }
```


#### 测试

可通过上述注解中的rtmp地址进行测试

**推流**
```
ffmpeg -re -i 35.flv  -vcodec copy -f flv rtmp://192.168.3.20:1935/live/test
```
![](/images/06_rtmp/0.png)
**直播**
```
ffplay rtmp://192.168.3.20:1935/live/test
```
![](/images/06_rtmp/1.png)

**点播**
```
ffplay rtmp://192.168.3.20:1935/record/test-1558006517-20190516_193517.flv
```
![](/images/06_rtmp/2.png)
