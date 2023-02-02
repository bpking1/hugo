---
title: surface pro4安装官方chrome OS及windows10双系统
date: 2020/08/26 0:46:49
---

## 需求

​		如果不是将surfece当作主力机的话，windows实在过于臃肿，相比之下，chromeOS使用起来更加流畅，省电，还兼容部分安卓app，触屏支持也更加完善，chromeOS有一个国内本土化版本[Fyde OS](https://fydeos.com/)  ，跟官方版相比，FydeOS采用的是定制版浏览器，不需要登陆google账号，好处是不用爬墙，缺点也很明显，没办法同步插件、密码等等，此外FydeOS的surfece pro4版本的双系统引导有问题，经常性的引导不了，尝试google play也安装不上，后来发现github上的[brunch](https://github.com/sebanc/brunch) 这个项目，在这个项目的帮助下安装上了官方chromeOS。

## 安装步骤

#### 		1.下载ChromeOS官方恢复镜像

​		surfece pro4要下载代号为rammus的镜像，目前brunch最新版支持到83版本，所以我这里下载的是[83版本](https://dl.google.com/dl/edgedl/chromeos/recovery/chromeos_13020.87.0_rammus_recovery_stable-channel_mp-v2.bin.zip) 

#### 		2.下载brunch

​		在项目的[github release](https://github.com/sebanc/brunch/releases)中下载，我这里下载的是最新版，支持到chromeOS83版本，注意brunch的版本需要和官方		恢复镜像版本一致

#### 		3.在windows商店中安装Ubuntu子系统 WSL(其他linux虚拟机也可以)

#### 		4.启动虚拟机，并安装必要的工具

```
 sudo apt update && sudo apt install pv tar cgpt
```

#### 		5.进入brunch和chromeOS恢复镜像的下载目录，解压brunch压缩包

```
cd /mnt/c/Users/< username >/Downloads/
sudo tar zxvf brunch_< version >.tar.gz
```

#### 		6.制作chromeOS系统镜像，注意磁盘空间至少需要剩余14g

```
sudo bash chromeos-install.sh -src 替换为chromeOS官方恢复镜像的路径 -dst chromeos.img
```

​		执行成功后就得到一个名为chromeos.img的chromeOS系统镜像，这里放一个我已经制作好了的img[下载链接](https://dotnetcore-my.sharepoint.com/:u:/g/personal/hejie_dotnetcore_onmicrosoft_com/EfhTvrtRIX1Eh9RLhnZg0CUBMUzJXjZFj_dhke6QQBQzkA?e=dZkrg0)，对应的是目前最新brunch和chromeOS83版本，如果不想自己制作可以跳过上面的步骤

#### 		7.制作u盘启动盘

​		推荐使用[balenaEtcher](https://www.balena.io/etcher/) 来制作，简单明了

​		![image-20200825224742283](https://raw.githubusercontent.com/jerry1119/blogPics/master/img/image-20200825224742283.png)

​		第一步选择刚才制作好的chromeos.img ，第二步选择你的u盘，第三部点击Flash等待几分钟即可，

​		制作完成后，你的u盘会被windows识别为多个盘，不用管，以后想恢复的话，用驱动精灵删掉所有分区并重新分区      格式化即可

#### 		8.关闭磁盘加密

​		左下角搜索栏输入BitLocker，如果为开启状态，将其关闭

​		![image-20200825234914784](https://raw.githubusercontent.com/jerry1119/blogPics/master/img/image-20200825234914784.png)

#### 		9.在windows中为chromeOS新建一个硬盘分区

​		进入磁盘管理，右键选择压缩卷，分配大小至少为14g以上，建议30g左右，对分配出来的分区右键选择新建简单卷，格式选择NTFS

![image-20200825232446427](https://raw.githubusercontent.com/jerry1119/blogPics/master/img/image-20200825232446427.png)

#### 		10.进入UEFI关闭surface pro安全启动

​		长按音量+键和电源键，大约十几秒，即可进入UEFI设置菜单

<img src="https://raw.githubusercontent.com/jerry1119/blogPics/master/img/4560390_en_1" alt="Surface UEFI 安全屏幕的屏幕截图。" style="zoom:150%;" />

​		在Secure Boot这里关闭安全启动

#### 		11.将u盘插到surface上，选择从u盘启动

​		依次进入windows设置->恢复->高级启动->立即重新启动，然后选择从u盘启动

![image-20200825231030328](https://raw.githubusercontent.com/jerry1119/blogPics/master/img/image-20200825231030328.png)

​		这个时候chromeOS已经安装在u盘中，可以使用了，但是想要直接将它安装在硬盘中，还需要额外步骤		

#### 		12.将chromeOS安装到硬盘上

​		在chromeOS中打开 shell (CTRL+ALT+T打开crosh，输入shell回车)

![image-20200826020712142](https://raw.githubusercontent.com/jerry1119/blogPics/master/img/image-20200826020712142.png)

​		输入lsblk，可以看到，我这里之前为chromeOS分配出来的30g硬盘空间名为nvme0n1p4

​		挂载这个分区用来作为chromeOS的启动盘		

```
mkdir -p ~/tmpmount
sudo mount 改为为chromeOS建立的硬盘分区位置 ~/tmpmount
```

​		比如我这里就应该为 sudo mount  /dev/nvme0n1p4 ~/tmpmount

​		然后执行如下命令将chromeOS安装到硬盘分区中

```
sudo bash chromeos-install -dst ~/tmpmount/chromeos.img -s 30
```

​		后面的30是，为chromeOS分配的大小30g，根据实际情况修改

​		运行结束后会生成GRUB配置信息，将其复制下来保存

#### 	  13.配置GRUB双系统引导

​		进入windows，下载[Grub2Win](https://techseedr.wixsite.com/website/post/grub2win-standalone-offline-installer) 引导工具，安装后点击最下面Manage Boot Menu ，然后点击Add A New Entry ，然后Type选择submenu，Title改为ChromeOS，然后点击Edit Custom Code，将之前保存下来的Grub信息复制进去，点击apply，apply关闭。重启会进入Grub引导界面，选择ChromeOS即可进入

## 其他问题

​		某些surface pro4触屏会[偶尔失灵](https://github.com/sebanc/brunch/issues/336)，需要在grub信息中，linux那一行的结尾添加 intel-ipts.no_feedback=1 即可

​		windows和chromeOS协作问题，window之间可以通过微软官方的Mouse without Borders来共享键鼠，但是chromeOS目前还没有能支持共享键鼠的软件，Synergy虽然跨平台支持linux，但是[不支持](https://chromium.googlesource.com/chromiumos/docs/+/master/containers_and_vms.md#Will-synergy-work)chromeOS。

​		chrome有插件远程桌面使用起来不是很流畅，最好的解决办法有两种，一是kvm切换器，切换起来比较麻烦，二是使用罗技优联键鼠，而且可以自己改造机械键盘加入罗技优联模块，参考张大妈上的[文章](https://post.smzdm.com/p/awxlw5dk/) ，这应该是最完美的方案

​	参考资料：https://github.com/sebanc/brunch 

​					  https://www.youtube.com/watch?v=kyBwqis91MM

