---
title: emby挂载网盘转直链播放
date: 2021/09/09 22:15:00
---

将emby视频播放地址劫持到直链网盘,下面以onemanager为例,别的盘自己参考修改
onemanager 地址 : https://github.com/qkqpttgf/OneManager-php
## 1.
```
wget https://pan.bpking.ml/e5main/other/emby/embyOnemanager.tar && mkdir -p ~/embyOnemanager && tar -xvf ./embyOnemanager.tar -C ~/embyOnemanager && cd ~/embyOnemanager
```
## 2. 
将conf.d/emby.js 中的api_key 修改为 自己emby的api_key
获取方法：emby server控制台 --》高级 --》Api密钥 自己设置一个

## 3. 
启动服务:
```bash
docker-compose up -d
```
查看启动log:
```bash
docker-compose logs -f
```

防火墙放行8095 和 8288端口
访问8288端口为onemanager 端口, 8095端口为emby转直链端口与默认的8096互不影响

## 4.
使web身份可读写onemanager代码中的.data/config.php文件，推荐chmod 666 .data/config.php。

在8288端口 onemanager挂载各种网盘,盘名必须与 rclone挂载的文件夹名相同
例如rclone挂载文件夹名为 :  /mnt/ali      /mnt/onedrive   /mnt/gd  /mnt/sharepoint
那么这几个盘在onemanager中以同样的标签名字命名  ali   onedrive  gd  sharepoint

注: onemanager至少要挂2个盘才行,只有一个盘的话,标签名会被省略,导致直链失败
onemanager 目录如果设置了密码的话, 需要在设置里面允许 不用密码也能下载 , 即: downloadencrypt 这一项 填  1 

## 5. 
直链播放不支持转码,转码的话只能走emby server
所以最好 在emby控制台将 用户的 转码权限关掉,确保走直链

访问 8095端口打开emby 测试直链是否生效

8095端口为走直链端口  , 原本的 8096端口 走 emby server 不变

## 已知问题 
onemanager好像不支持 阿里盘和 gd盘的直链..


## 补充:  在原基础上增加反代到goindex支持

项目地址 : https://github.com/Achrou/goindex-theme-acrou
修改 emby.js 文件  将 .then(uri => r.internalRedirect(uri))   这一行 改为:

```
.then(uri => uri.includes('bpking')? r.return(301, uri.replace('/mnt/bpking/','https://gd.fgnb.icu/1:/')) : r.internalRedirect(uri))
```
其中 bpking 为我的 gd盘挂载的文件夹名  路径为 /mnt/bpking/
https://gd.fgnb.icu/1:/    为 这个盘在goindex 中根目录的网页地址栏上面的地址
根据自己的情况  修改 这3个地方 

例如: 原本一部视频在emby 中媒体信息看到路径是   /mnt/bpking/电影/汪汪队立大功/汪汪队立大功.mkv
而在 goindex 中的直链路径为 https://gd.fgnb.icu/1:/电影/汪汪队立大功/汪汪队立大功.mkv
上面js代码的效果就是 将原本rclone挂载路径替换为了goindex直链路径 

修改之后记得执行 重启nginx 使修改生效
```bash
docker restart emby-nginx
```


挂载多个goindex盘的话,请参考下面的代码修改 emby.js文件:
```js
.then(path => path.includes('pan1')? path.replace('/mnt/pan1/','https://gd.fgnb.icu/1:/'):path)
                .then(path => path.includes('pan2')? path.replace('/mnt/pan2/','https://gd.fgnb.icu/2:/'):path)
                .then(path => path.includes('pan3')? path.replace('/mnt/pan3/','https://gd.fgnb.icu/3:/'):path)
                //.then(uri => encodeURI(uri))
                .then(uri => uri.includes('https://gd.fgnb.icu')? r.return(301, uri) : r.internalRedirect(uri))
                .catch(e => r.return(501, e.message));
```

其中 pan1  pan2  pan3  这三个盘 与 goindex 网页链接前缀对应  xxx/1:   xxxx/2:  xxxx/3:

注:  goindex 需要设置为 不保护文件链接 "protect_file_link": false  ,在 cf worker的代码里面修改

​      文档中有错误的地方或者有好的方法请联系 @baipiaoking 修改,谢谢

