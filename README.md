# Nginx-RTMP流媒体服务器搭建【centos 7】

### 流媒体是什么？

RTMP是一种设计用来进行实时数据通信的网络协议，主要用来在Flash/AIR平台和支持RTMP协议的流媒体/交互服务器之间进行音视频和数据通信。是目前主流的流媒体传输协议，广泛用于直播领域。以流方式在网络中传送音频、视频和多媒体文件的媒体形式。流媒体的典型特征是把连续的音频和视频信息压缩后放到网络服务器上，用户可以边下载边观看，不需要等待整个文件缓冲完。

### 开发环境
```
linux:centos 7
Nginx:1.14.2
nginx-rtmp-module：1.2.1
```

### 安装编译扩展
```
yum install automake autoconf make gcc gcc-c++ 

yum install build-essential libpcre3 libpcre3-dev libssl-dev

yum install -y epel-release

sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

rpm --import  http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro

rpm -Uvh  http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm


```


### 下载nginx-rtmp-module
```
wget https://github.com/arut/nginx-rtmp-module/archive/v1.2.1.tar.gz

tar -zxvf nginx-rtmp-module-1.2.1.tar.gz
```


### 编译安装nginx
```
wget http://nginx.org/download/nginx-1.14.2.tar.gz

解压：
    tar -zxvf nginx-1.14.2.tar.gz

cd nginx-1.14.2

编译：
    ./configure --with-http_ssl_module --add-module=[nginx-rtmp-module的存放路径]
安装：
    make && make install
```

### 安装
```
yum install -y ffmpeg

查看版本：
    ffmpeg -version
```

### nginx.conf 配置
```
rtmp {                #RTMP服务
    server {
       listen 1935;  #//服务端口
       chunk_size 4096;   #//数据传输块的大小
       application vod {
           play /opt/video; #//视频文件存放位置。
       }
       application rtmplive {
          live on;
          #为 rtmp 引擎设置最大连接数。默认为 off
          max_connections 1024;
       }
       application live{ #直播
           live on;
           hls on; #这个参数把直播服务器改造成实时回放服务器。
           wait_key on; #对视频切片进行保护，这样就不会产生马赛克了。
           hls_path /opt/video/hls; #切片视频文件存放位置。
           hls_fragment  600s;     #设置HLS片段长度。
           hls_playlist_length 10m;  #设置HLS播放列表长度，这里设置的是10分钟。
           hls_continuous on; #连续模式。
           hls_cleanup on;    #对多余的切片进行删除。
           hls_nested on;     #嵌套模式。
       }
   }
}
```

### 开启防火墙端口
```
查看以开放端口：
firewall-cmd --list-ports

开放tcp8080，80端口
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --zone=public --add-port=80/tcp --permanent
```


### 播放源
```
推流：
rtmp://192.168.1.163/live/
拉流：
rtmp://192.168.1.163/live360p/test 
视频播放：
rtmp://ip:port/vod/test.flv
```

