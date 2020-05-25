### 时间如果不正常，超正常时间8小时
```bash
timedatectl set-local-rtc 1 --adjust-system-clock
timedatectl set-ntp 0
```
### 更新镜像 && 更新数据源
```bash
sudo pacman-mirrors -i -c China -m rank
```
### 排列源
```bash
sudo pacman-mirrors -g
sudo pacman-optimize && sync (固态硬盘直接忽略)
```
### 更新数据库并且升级系统  &&　顺便安装yay
```bash
sudo pacman -Syyu
sudo pacman -S yay
```
### 直接安装emacs或者vim
```bash
yay -S vim
yay -S clang && yay -S emacs
```
### 添加中科大源
```bash
sudo emacs -nw /etc/pacman.conf

[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```
### 导入GPG Key:
```bash
yay -Syy && yay -S archlinuxcn-keyring
```
### 安装搜狗拼音输入法
```bash
yay -S fcitx-im && yay -S fcitx-configtool
yay -S fcitx-sogoupinyin
sudo emacs -nw ~/.xprofile

export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```
### 搜狗输入法有问题解决方法 [https://blog.csdn.net/kfeng632/article/details/90513211](https://blog.csdn.net/kfeng632/article/details/90513211)
```bash
sogou-qimpanel: error while loading shared libraries: libfcitx-qt.so.0: cannot open shared object file: No such file or directory

yay -S fcitx-qt4
```

### 中文乱码
```bash
yay -S wqy-microhei
```

### V2ray安装
https://github.com/v2ray/manual/blob/master/zh_cn/chapter_00/install.md
```bash
bash <(curl -L -s https://install.direct/go.sh)
```
### V2ray配置
https://github.com/v2ray/manual/blob/master/zh_cn/chapter_00/start.md
```json
{
  "inbounds": [{
    "port": 1080,  // SOCKS 代理端口，在浏览器中需配置代理并指向这个端口
    "listen": "127.0.0.1",
    "protocol": "socks",
    "settings": {
      "udp": true
    }
  }],
  "outbounds": [{
    "protocol": "vmess",
    "settings": {
      "vnext": [{
        "address": "server", // 服务器地址，请修改为你自己的服务器 ip 或域名
        "port": 10086,  // 服务器端口
        "users": [{ "id": "b831381d-6324-4d53-ad4f-8cda48b30811" }]
      }]
    }
  },{
    "protocol": "freedom",
    "tag": "direct",
    "settings": {}
  }],
  "routing": {
    "domainStrategy": "IPOnDemand",
    "rules": [{
      "type": "field",
      "ip": ["geoip:private"],
      "outboundTag": "direct"
    }]
  }
}
```

### manjaro install deepin.com.qq.office
1、添加archlinuxcn
```bash
[archlinuxcn]
SigLevel = Optional TrustAll
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

2、安装deepin.com.qq.office
```bash
sudo pacman -S deepin.com.qq.office
```

3、乱码问题
下载[simsun.ttc](https://github.com/sonatype/maven-guide-zh/blob/master/content-zh/src/main/resources/fonts/simsun.ttc)
```bash
mv simsun.ttc /home/serical/.deepinwine/Deepin-TIM/drive_c/windows/Fonts
```

4、解决收缩问题topicons，menu->tweaks->extensions->topicons plus on
[https://extensions.gnome.org/extension/1031/topicons/](https://extensions.gnome.org/extension/1031/topicons/)  
安装google扩展，点击on，


## 安装MySQL8问题
### 1、[libprotobuf-lite.so.18](https://learnku.com/articles/36333?order_by=created_at&)问题
```sh
https://github.com/protocolbuffers/protobuf/releases/tag/v3.7.1
protobuf-all-3.7.1.tar.gz
./autogen.sh
./configure
make
make install

➜  ~ ll /usr/lib/libprotobuf-lite.so*
lrwxrwxrwx 1 root root   26 11月 15 20:56 /usr/lib/libprotobuf-lite.so -> libprotobuf-lite.so.21.0.0
-rwxr-xr-x 1 root root 5.8M 11月 15 20:46 /usr/lib/libprotobuf-lite.so.18
lrwxrwxrwx 1 root root   26 10月  7 02:28 /usr/lib/libprotobuf-lite.so.21 -> libprotobuf-lite.so.21.0.0
-rwxr-xr-x 1 root root 734K 10月  7 02:28 /usr/lib/libprotobuf-lite.so.21.0.0

# 然后将 so 文件移动到 /usr/lib 目录下即可
cd /usr/lib/
cp /usr/local/lib/libprotobuf-lite.so.18.0.1 .

# 重命名为MySQL需要的libprotobuf-lite.so.18
mv libprotobuf-lite.so.18.0.1 libprotobuf-lite.so.18
```

### 2、mysqld --initialize --user=mysql --basedir=/usr --datadir=/var/lib/mysql,一定要用**sudo**执行,不然各种错误
```sh
:: You need to initialize the MySQL data directory prior to starting
   the service. This can be done with mysqld --initialize command, e.g.:
   mysqld --initialize --user=mysql --basedir=/usr --datadir=/var/lib/mysql
:: Additionally you should secure your MySQL installation using
   mysql_secure_installation command after starting the mysqld service

➜  ~ sudo mysqld --initialize --user=mysql --basedir=/usr --datadir=/var/lib/mysql
2019-11-15T14:02:16.228936Z 0 [Warning] [MY-010915] [Server] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2019-11-15T14:02:16.228974Z 0 [System] [MY-013169] [Server] /usr/bin/mysqld (mysqld 8.0.17) initializing of server in progress as process 6772
2019-11-15T14:02:18.498640Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: x1DA1ert#hiD
2019-11-15T14:02:20.563492Z 0 [System] [MY-013170] [Server] /usr/bin/mysqld (mysqld 8.0.17) initializing of server has completed

# 进入MySQL
➜  ~ mysql -uroot -p'x1DA1ert#hiD'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.17

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
# 修改密码
mysql> alter user root@localhost identified by '88888888';
Query OK, 0 rows affected (0.02 sec)

# 创建数据库
create database knowledge_pay
    default character set utf8mb4
    default collate utf8mb4_general_ci;
```

### 3、MySQL版本降级 [archlinux如何降级安装软件包](http://blog.lujun9972.win/blog/2018/10/15/archlinux%E5%A6%82%E4%BD%95%E9%99%8D%E7%BA%A7%E5%AE%89%E8%A3%85%E8%BD%AF%E4%BB%B6%E5%8C%85/index.html)
```
# 找到缓存中 的上一个版本
ll /var/cache/pacman/pkg/mysql-8.0.17-2-x86_64.pkg.tar.xz
ll /var/cache/pacman/pkg/mysql-clients-8.0.17-2-x86_64.pkg.tar.xz

# 降级
sudo pacman -U /var/cache/pacman/pkg/mysql-8.0.17-2-x86_64.pkg.tar.xz
sudo pacman -U /var/cache/pacman/pkg/mysql-clients-8.0.17-2-x86_64.pkg.tar.xz
```

### 4 MySQL忽略版本更新
```
# 查询MySQL包名
➜  ~ pacman -Q mysql
mysql 8.0.17-2
➜  ~ pacman -Q mysql-clients 
mysql-clients 8.0.17-2

# 忽略配置
➜  ~ sudo vim /etc/pacman.conf
IgnorePkg   = mysql mysql-clients
```

## teamviewer 未就绪 网络问题
参考[Teamviewer9 cannot start (PID file not readable)](https://bbs.archlinux.org/viewtopic.php?id=205190)
```sh
➜  ~ sudo rm /opt/teamviewer/config/global.conf
# 再重新打开teamviewer即可
```