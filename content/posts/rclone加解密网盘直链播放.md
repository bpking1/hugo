---
title: rclone加解密网盘emby直链播放
slug: emby_jellyfin_to_rclone_encrypt_directlink
date: "2023-4-11"
draft: false
categories:
    - fun
tags:
    - emby
    - rclone
    - jellyfin
    - encrypt
deescription: rclone encrypt for emby/jellyfin direct link
---

## 使用场景：
国内网盘有审查机制会屏蔽掉一些资源，因此使用rclone encrypt加密文件上传到网盘来避免和谐,rclone的加解密使用的是XSalsa20加密算法(chacha20的变种)，实测6g的阿里云盘视频解密播放，拖动大概需要等1-2s时间

在此基础上搭建jellyfin/emby,并实现网盘直链播放，直链播放具体设置需要参考之前的文章

## 实现思路：
因为使用了rclone的加密模块，所以播放的时候必须要rclone解密才行，正好rclone 有一个serve功能，可以将rclone的网盘提供http服务来访问。所以可以在播放端运行rclone serve http, emby/jellyfin服务端将视频播放地址重定向到播放端的rclone http server,以实现网盘直链解密播放

## 主要流程：
1. 使用rclone设置网盘加密目录
   参考官方文档 https://rclone.org/crypt/

2. 将资源上传到rclone的加密目录

3. 将rclone加密目录挂载到硬盘，emby/jellyfin扫描这个目录

4. 修改直链的emby.js文件
添加如下代码，这里的 secretPath 是指rclone加密目录的挂载路径，看情况修改
http://127.0.0.1:28080 是指 播放端 rclone http server的地址
```js
  r.warn(`mount emby file path: ${embyRes}`);
    const secretPath = '/mnt/secret-alist'; 
    if(embyRes.includes(secretPath)) {
	const localpath = embyRes.replace(secretPath, 'http://127.0.0.1:28080');
        r.warn(`redirect to: ${localpath}`);
        r.return(302, localpath);
        return;
	
    }

    //fetch alist direct link
```
修改之后记得 nginx -s reload重启 nignx

5. 播放端安装rclone
以安卓手机为例,安装termux,将rclone的配置文件复制到termux的 ~/.config/rclone目录下
```bash
pkg update
pkg install rclone
然后运行rclone http server, 这里的 sercret: 是指rclone的加密盘名字
rclone serve http --addr 127.0.0.1:28080 secret:  --header "Referer:"
```
此时手机浏览器打开 127.0.0.1:28080 应该能看到加密盘的内容
rclone 官方文档:  https://rclone.org/commands/rclone_serve_http/

6. emby/jellyfin播放rclone加密目录的视频，即可实现直链解密播放

## 已知问题:
网页无法播放，尽量使用外部播放器播放
