---
title: 利用Docker挂载Nginx-rtmp(服务器直播流分发)+FFmpeg(推流)+Vue.js结合Video.js(播放器流播放)来实现实时网络直播
date: 2020-04-19 09:55:54
tags: [vue, live]
---
转载自 <https://v3u.cn/book/zhibo.html>

众所周知，在视频直播领域，有不同的商家提供各种的商业解决方案，其中比较靠谱的服务商有阿里云直播，腾讯云直播，以及又拍云和网易云的有偿直播服务，服务包括软硬件设备，摄像机，编码器，流媒体服务器等。但是其高昂的费用以及较高的准入门槛让许多个人和小型企业望而却步，本文要讲解的是如何使用nginx-rtmp搭建直播服务器，配合FFmpeg推流，在网页端vue.js作为载体利用video.js作为流播放器，打造一套可用的在线视频直播方案。

搭建直播服务器是一个漫长而复杂的过程，编译设置有点繁琐。好在docker上有大把别人编译设置好的rtmp环境，所以可以直接拿来用，docker的优越性由此可见一斑，这里用到的是alfg/nginx-rtmp库。

安装docker，可以在阿里云的镜像上下载比较快一点 <http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/>

目前稳定版是DockerToolbox-18.03.0-ce.exe 直接点击安装即可
安装过程有一点需要注意，必须要开启本机bios的cpu虚化技术

安装好docker后，下载nginx-rtmp镜像，并且运行服务映射端口到1945和8000

```bash
docker pull alfg/nginx-rtmp
docker run -it -p 1935:1935 -p 8000:80 --rm alfg/nginx-rtmp
```

访问宿主的8000端口显示nginx欢迎页面

{% asset_img nginx.png %}

然后利用FFmpeg进行推流操作，ffmpeg是什么请移步：[Python3利用ffmpeg针对视频进行一些操作](https://v3u.cn/a_id_74)

输入命令，注意摄像头和麦必须和电脑的设备吻合，另外推流服务器也要推到刚刚部署好的nginx上面去

```bash
ffmpeg -f dshow -i video="VMware Virtual USB Video Device":audio="Microphone (High Definition Audio Device)" -tune:v zerolatency -f flv "rtmp://192.168.99.100:1935/stream/test"
```

{% asset_img device.png %}

推流成功后，我们就要在网站上观看现场直播了，这里前端服务我们使用vue.js来搭建，视频流播放器我们使用video.js

首先建立一个直播的脚手架项目，然后安装一下必要的直播库，最后启动项目

```bash
#建立项目
vue init webpack-simple zhibo
cd zhibo
yarn install vue-router save
yarn install

#安装直播组件
yarn install video.js
yarn install aes-decrypter
yarn install m3u8-parser
yarn install mpd-parser
yarn install mux.js
yarn install url-toolkit
yarn install videojs-contrib-hls

#热启动项目
yarn run dev
```

新建一个video.vue，添加播放代码

```html
<template>
    <div>
      <video id="my-video" class="video-js vjs-default-skin" controls preload="auto" >
          <!-- 直播地址就是nginx映射后的播放地址，注意后缀为直播流的m3u8 -->
          <source src="http://192.168.99.100:8000/live/test.m3u8" >
      </video>
    </div>
</template>

<script>
import videojs from 'video.js'
import 'videojs-contrib-hls'

export default {
  data () {
    return {}
  },
  mounted() {
    videojs('my-video', {
      bigPlayButton: true,
      textTrackDisplay: false,
      posterImage: true,
      errorDisplay: false,
      controlBar: true
    }, function () {
      this.play()
    })
  }
}
</script>

<style>
</style>
```

最后，在vue.js的路由里绑定video.vue组件，进入浏览器，测试直播播放
