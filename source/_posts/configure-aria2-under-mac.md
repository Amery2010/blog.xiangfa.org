---
title: Mac 下配置 Aria2
author: 子丶言
ignoreCopyright: true
date: 2021-12-22 21:00:49
tags: ['Aria2', 'Mac']
categories: ['Aria2']
---

Aria2 是一款自由、跨平台命令行界面的下载管理器，该软件根据 GPLv2 许可证进行分发。支持的下载协议有：HTTP、HTTPS、FTP、Bittorrent 和 Metalink。
<!-- more -->

## 安装和设置 Aria2

```bash
# 使用 Homebrew 安装 aria2
brew install aria2

# 创建配置文件aria2.conf和空对话文件aria2.session
mkdir ~/.aria2 && cd ~/.aria2
touch aria2.conf
touch aria2.session
```

编辑配置文件`aria2.conf`

```bash
## '#'开头为注释内容, 选项都有相应的注释说明, 根据需要修改 ##
## 被注释的选项填写的是默认值, 建议在需要修改时再取消注释  ##

## 文件保存相关 ##

# 文件的保存路径(可使用绝对路径或相对路径), 默认: 当前启动位置
dir=${HOME}/Downloads
# 启用磁盘缓存, 0为禁用缓存, 需1.16以上版本, 默认:16M
disk-cache=32M
# 文件预分配方式, 能有效降低磁盘碎片, 默认:prealloc
# 预分配所需时间: none < falloc ? trunc < prealloc
# falloc和trunc则需要文件系统和内核支持
# NTFS建议使用falloc, EXT3/4建议trunc, MAC 下需要注释此项
#file-allocation=none
# 断点续传
continue=true

## 下载连接相关 ##

# 最大同时下载任务数, 运行时可修改, 默认:5
#max-concurrent-downloads=5
# 同一服务器连接数, 添加时可指定, 默认:1, 最大值16
max-connection-per-server=5
# 最小文件分片大小, 添加时可指定, 取值范围1M -1024M, 默认:20M
# 假定size=10M, 文件为20MiB 则使用两个来源下载; 文件为15MiB 则使用一个来源下载
min-split-size=10M
# 单个任务最大线程数, 添加时可指定, 默认:5
#split=5
# 分片选择算法,有助于视频的边下边播同时兼顾减少建立连接的次数
stream-piece-selector=geom
# 整体下载速度限制, 运行时可修改, 默认:0
#max-overall-download-limit=0
# 单个任务下载速度限制, 默认:0
#max-download-limit=0
# 整体上传速度限制, 运行时可修改, 默认:0
#max-overall-upload-limit=0
# 单个任务上传速度限制, 默认:0
#max-upload-limit=0
# 禁用IPv6, 默认:false
#disable-ipv6=true
# 连接超时时间, 默认:60
timeout=60
# 最大重试次数, 设置为0表示不限制重试次数, 默认:5
max-tries=5
# 设置重试等待的秒数, 默认:0
#retry-wait=0

## 进度保存相关 ##

# 日志文件
log-level=notice
log=${HOME}/.aria2/aria2.log
# 从会话文件中读取下载任务
# 需提前创建一个空文件否则会报错
input-file=${HOME}/.aria2/aria2.session
# 在Aria2退出时保存`错误/未完成`的下载任务到会话文件
save-session=${HOME}/.aria2/aria2.session
# 定时保存会话, 0为退出时才保存, 需1.16.1以上版本, 默认:0
save-session-interval=60

# 强制保存会话, 即使任务已经完成, 默认:false
# 较新的版本开启后会在任务完成后依然保留.aria2文件
#force-save=true

## RPC相关设置 ##

# 启用RPC, 默认:false
enable-rpc=true
# 允许所有来源, 默认:false
rpc-allow-origin-all=true
# 允许非外部访问, 默认:false
rpc-listen-all=true
# RPC监听端口, 端口被占用时可以修改, 默认:6800
rpc-listen-port=6800
# 设置的RPC授权令牌
# 此处使用`openssl rand -base64 32`命令生成<TOKEN>
rpc-secret=<TOKEN>
# 是否启用 RPC 服务的 SSL/TLS 加密,
# 启用加密后 RPC 服务需要使用 https 或者 wss 协议连接
#rpc-secure=true
# 在 RPC 服务中启用 SSL/TLS 加密时的证书文件,
# 使用 PEM 格式时，您必须通过 --rpc-private-key 指定私钥
#rpc-certificate=/path/to/certificate.pem
# 在 RPC 服务中启用 SSL/TLS 加密时的私钥文件
#rpc-private-key=/path/to/certificate.key

## HTTP 设置 ##

# 自定义 User Agent
user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.85 Safari/537.36

## BT/PT下载相关 ##

# 当下载的是一个种子(以.torrent结尾)时, 自动开始BT任务, 默认:true
follow-torrent=true
# BT监听端口, 当端口被屏蔽时使用, 默认:6881-6999
listen-port=6881-6999
# 单个种子最大连接数, 默认:55
#bt-max-peers=55

### DHT 功能, 仅对 BT 生效, PT 无效###
# 打开 DHT (IPv4) 功能
enable-dht=true
# 打开 DHT (IPv6) 功能
enable-dht6=true
# DHT网络监听端口, 默认:6881-6999
dht-listen-port=6881-6999
# 本地节点查找
bt-enable-lpd=true
# 种子交换
enable-peer-exchange=true
# DHT (IPv4) 路由表文件路径
dht-file-path=${HOME}/.aria2/dht.dat
# DHT (IPv6) 路由表文件路径
dht-file-path6=${HOME}/.aria2/dht6.dat

# 客户端伪装, PT需要
peer-id-prefix=-UT341-
peer-agent=uTorrent/341(109279400)(30888)

# 同一服务器连接数
# 每个种子限速, 对少种的PT很有用, 默认:50K
#bt-request-peer-speed-limit=50K
# 当种子的分享率达到这个数时, 自动停止做种, 0为一直做种, 默认:1.0
seed-ratio=0
# BT校验相关, 默认:true
#bt-hash-check-seed=true
# 继续之前的BT任务时, 无需再次校验, 默认:false
bt-seed-unverified=true
# 保存磁力链接元数据为种子文件(.torrent文件), 默认:false
bt-save-metadata=true

# BT 服务器地址
# 逗号分隔的 BT 服务器地址. 如果服务器地址在 --bt-exclude-tracker 选项中, 其将不会生效.
bt-tracker=udp://tracker.coppersurfer.tk:6969/announce,udp://tracker.leechers-paradise.org:6969/announce,udp://tracker.opentrackr.org:1337/announce,udp://p4p.arenabg.com:1337/announce,udp://9.rarbg.to:2710/announce,udp://9.rarbg.me:2710/announce,udp://tracker.internetwarriors.net:1337/announce,udp://exodus.desync.com:6969/announce,udp://tracker.tiny-vps.com:6969/announce,udp://tracker.moeking.me:6969/announce,udp://retracker.lanta-net.ru:2710/announce,udp://open.stealth.si:80/announce,udp://open.demonii.si:1337/announce,udp://tracker.torrent.eu.org:451/announce,udp://tracker.cyberia.is:6969/announce,udp://denis.stalker.upeer.me:6969/announce,udp://tracker3.itzmx.com:6961/announce,udp://ipv4.tracker.harry.lu:80/announce,udp://valakas.rollo.dnsabr.com:2710/announce,udp://tracker.nyaa.uk:6969/announce,udp://retracker.netbynet.ru:2710/announce,udp://opentor.org:2710/announce,udp://explodie.org:6969/announce,http://explodie.org:6969/announce,udp://zephir.monocul.us:6969/announce,udp://xxxtor.com:2710/announce,udp://tracker.zum.bi:6969/announce,udp://tracker.yoshi210.com:6969/announce,udp://tracker.uw0.xyz:6969/announce,udp://tracker.sbsub.com:2710/announce,udp://tracker.lelux.fi:6969/announce,udp://tracker.iamhansen.xyz:2000/announce,udp://tracker.filemail.com:6969/announce,udp://tracker.dler.org:6969/announce,udp://retracker.sevstar.net:2710/announce,udp://retracker.akado-ural.ru:80/announce,udp://open.nyap2p.com:6969/announce,udp://chihaya.toss.li:9696/announce,udp://bt2.archive.org:6969/announce,udp://bt1.archive.org:6969/announce,udp://bt.okmp3.ru:2710/announce,https://tracker.nanoha.org:443/announce,http://tracker.torrentyorg.pl:80/announce,http://tracker.opentrackr.org:1337/announce,http://tracker.internetwarriors.net:1337/announce,http://tracker.bt4g.com:2095/announce,http://t.nyaatracker.com:80/announce,http://retracker.sevstar.net:2710/announce,http://pow7.com:80/announce,http://mail2.zelenaya.net:80/announce,http://h4.trakx.nibba.trade:80/announce,udp://tracker4.itzmx.com:2710/announce,udp://tracker2.itzmx.com:6961/announce,udp://tracker.zerobytes.xyz:1337/announce,udp://tracker.swateam.org.uk:2710/announce,udp://tr.bangumi.moe:6969/announce,udp://qg.lorzl.gq:2710/announce,udp://opentracker.i2p.rocks:6969/announce,udp://bt2.54new.com:8080/announce,https://tracker.opentracker.se:443/announce,https://tracker.lelux.fi:443/announce,http://www.loushao.net:8080/announce,http://vps02.net.orel.ru:80/announce,http://tracker4.itzmx.com:2710/announce,http://tracker3.itzmx.com:6961/announce,http://tracker2.itzmx.com:6961/announce,http://tracker1.itzmx.com:8080/announce,http://tracker01.loveapp.com:6789/announce,http://tracker.zerobytes.xyz:1337/announce,http://tracker.yoshi210.com:6969/announce,http://tracker.nyap2p.com:8080/announce,http://tracker.lelux.fi:80/announce,http://tracker.bz:80/announce,http://opentracker.i2p.rocks:6969/announce,http://open.acgnxtracker.com:80/announce
# BT 排除服务器地址
bt-exclude-tracker=

# 启用后台进程
daemon=false

# 部分事件hook, 调用第三方命令:/path/to/command
# BT下载完成(如有做种将包含做种，如需调用请务必确定设定完成做种条件)
on-bt-download-complete=${HOME}/.aria2/download-complete-hook.sh
# 下载完成
on-download-complete=${HOME}/.aria2/download-complete-hook.sh
# 下载错误
on-download-error=

# 代理 仅支持 HTTP 协议
#all-proxy=http://127.0.0.1:1087
```

- [设置文件详解](#file-aria2-template-conf)
- [自用设置文件](#file-aria2-conf)

本人设置文件:

- 默认开启 RPC 模式
- 已设置RPC授权令牌, 详见设置文件注释
- 已经添加 BT tracker，更多详见 [XIU2/TrackersListCollection](https://trackerslist.com/#/zh)

## 设置为macOS的开机启动
> 参考: [控制macOS的开机启动](https://www.jianshu.com/p/eee8a7de179c)

### 创建用户启动文件

```bash
touch ~/Library/LaunchAgents/aria2.plist
```
写入如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>KeepAlive</key>
    <true/>
    <key>Label</key>
    <string>aria2</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/aria2c</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>WorkingDirectory</key>
    <string>${HOME}/Downloads</string>
</dict>
</plist>
```
> 注意: 修改`WorkingDirectory`目录

```bash
# 检查plist语法是否正确
plutil ~/Library/LaunchAgents/aria2.plist

# 修改文件权限
chmod 644 ~/Library/LaunchAgents/aria2.plist
```

### 添加并启用自启动项

```bash
# 添加自启动项: aria2
launchctl load ~/Library/LaunchAgents/aria2.plist

# 删除自启动项: aria2
launchctl unload ~/Library/LaunchAgents/aria2.plist

# 启动服务: aria2
launchctl start aria2

# 停止服务: aria2
launchctl stop aria2
```

> 更多`launchctl`使用方法, 详见命令手册<br>
> 可使用`killall aria2c` 结束进程, 并会自动重启进程

## 添加自动更新`BT tracker`功能

### 创建`trackers-list-aria2.sh`脚本
> 参考: [Aria2 bt-tracker跟踪服务器列表自动更新](https://www.feng.ee/aria2-trackers-auto-update.html)

脚本内容如下: 
```bash
#!/bin/bash
#trackers-list-aria2.sh
# aria2 设置文件路径
CONF=${HOME}/.aria2/aria2.conf

#设置选择的 trackerlist （可选 all_aria2.txt, best_aria2.txt, http_aria2.txt）
trackerfile=all_aria2.txt
#downloadfile=https://raw.githubusercontent.com/ngosang/trackerslist/master/${trackerfile}
downloadfile=https://trackerslist.com/${trackerfile}

list=$(curl -fsSL ${downloadfile})
if ! grep -q "bt-tracker" "${CONF}" ; then
    echo -e "\033[34m==> 添加 bt-tracker 服务器信息......\033[0m"
    echo -e "\nbt-tracker=${list}" >> "${CONF}"
else
    echo -e "\033[34m==> 更新 bt-tracker 服务器信息.....\033[0m"
    sed -i '' "s@bt-tracker.*@bt-tracker=${list}@g" "${CONF}"
fi

## 重启 aria2 服务
echo -e "\033[34m==> 停止 aria2 服务......\033[0m"
launchctl stop aria2
echo -e "\033[34m==> 启动 aria2 服务......\033[0m"
launchctl start aria2
```

脚本放置到 `~/.aria2/`，并设置运行权限:
```bash
chmod +x ~/.aria2/trackers-list-aria2.sh
```

### 设置任务计划程序 实现自动更新
> 参考:
> - [Aria2 bt-tracker跟踪服务器列表自动更新](https://www.feng.ee/aria2-trackers-auto-update.html)
> - [mac下crontab执行定时脚本](https://blog.csdn.net/ty_hf/article/details/72354230)

编译当前用户任务计划
```bash
crontab -e
```

在打开的`vi`中 键入如下, 并使用`:wq`命令保存退出, 可用`crontab -l`查看当前用户任务计划
```
0 18 * * * ~/.aria2/trackers-list-aria2.sh
```
或者 直接
```bash
(crontab -l 2&> /dev/null; echo "0 18 * * * ~/.aria2/trackers-list-aria2.sh") | crontab
```

> 以上表示: 每天下午 6 点自动更新`BT tracker`(并重启`aria2`服务)<br>
> 更多`crontab`时间的设定详见: [这里](https://user-images.githubusercontent.com/7850715/87239248-94674100-c43f-11ea-8445-1d084be61436.png)

> 取消计划任务
> ```bash
> crontab -e
> ```
> 然后手动删除, 或者
> ```bash
> crontab -l 2&> /dev/null| sed "/trackers-list-aria2.sh/d" | crontab
> ```

## 添加下载通知

> 参考: [macOS下 给aria2 RPC添加一个下载通知](https://github.com/maboloshi/Blog/blob/hexo/source/_posts/05.%20macOS%E4%B8%8B%20%E7%BB%99aria2%20RPC%E6%B7%BB%E5%8A%A0%E4%B8%80%E4%B8%AA%E4%B8%8B%E8%BD%BD%E9%80%9A%E7%9F%A5.md)

最终效果：当下载完成会在屏幕右上角弹出一个提示框显示具体下载完成的文件名，同时中文语音播报：“有个文件下载完成，请查收！”

![macOS 下aria2 提示框实例](https://user-images.githubusercontent.com/7850715/74525348-cc159f80-4f18-11ea-84bd-56be79bf3b0a.png)

### 创建`download-complete-hook.sh`脚本

> 参考:
> - [aria2 event-hook](https://aria2.github.io/manual/en/html/aria2c.html#event-hook)
> - [Display notification from the Mac command line](https://code-maven.com/display-notification-from-the-mac-command-line)
> - [在mac命令行执行显示通知](https://www.zixi.org/archives/notification_on_macos.html)
> - [Pass in variable from shell script to applescript](https://stackoverflow.com/a/17243326/7488424)

脚本内容如下: 
```bash
#!/bin/sh
# 给aria2 RPC添加一个下载完成通知 for macOS
# 最终效果：当下载完成会在屏幕右上角弹出一个提示框显示具体下载完成的文件名，
# 同时中文语音播报：“有个文件下载完成，请查收！”
# 变量 3 表示下载完成文件的路径
# 具体提示框设置可参考`https://code-maven.com/display-notification-from-the-mac-command-line`。
# 不支持设置自定义图标

fname=`basename $3`
osascript <<EOF
display notification "$fname 已经下载完成！" with title "【下载完成】"
say "有个文件下载完成，请查收！"
EOF
```

将脚本放置到 `~/.aria2/`，并设置运行权限:
```bash
chmod +x ~/.aria2/download-complete-hook.sh
```
### 添加 Hook 设置

> 参考:
> -  https://aria2.github.io/manual/en/html/aria2c.html#event-hook

在 aria2 设置文件`.aria2.conf`加入如下：

```bash
# BT下载完成(如有做种将包含做种，如需调用请务必确定设定完成做种条件)
on-bt-download-complete=${HOME}/.aria2/download-complete-hook.sh
# 下载完成
on-download-complete=${HOME}/.aria2/download-complete-hook.sh
```

## Aria2 web UI

无需安装，直接使用浏览器打开: [AriaNg版 UI](http://ariang.mayswind.net/latest/)

### PRC 设置
> 根据 aria2 配置文件中的 PRC 相关设置项进行设置

<img width="839" alt="AriaNg_PRC_设置" src="https://user-images.githubusercontent.com/7850715/73242932-3353f580-419e-11ea-8059-dc9490a5f595.png">

## 安装浏览器下载插件
[Aria2 for Chrome插件](https://chrome.google.com/webstore/detail/aria2-for-chrome/mpkodccbngfoacfalldjimigbofkhgjn)

- 内置一个离线 AriaNg版 UI
- 整合右键下载菜单

> 内置的离线 AriaNg版也需要设置PRC，否则无法“导出到 ARIA2 RPC”。

---
## **以下内容仅供参考 现阶段实用性不大**

## 百度云下载
> 方案来自: https://www.runningcheese.com/baiduyun

### 安装 [`网盘助手`](http://pan.newday.me/)

#### 浏览器插件版
直接进入[网盘助手](http://pan.newday.me/)主页,按浏览器不同下载并安装对应的插件

#### 脚本版
[网盘助手脚本](https://greasyfork.org/zh-CN/scripts/378301)，需要通过拓展 [Violentmonkey](https://violentmonkey.github.io/get-it/) 或者 [Tampermonkey](https://www.tampermonkey.net/) 等脚本管理器来启用

### 使用方法
> 配合[Aria2 for Chrome插件](https://chrome.google.com/webstore/detail/aria2-for-chrome/mpkodccbngfoacfalldjimigbofkhgjn)使用

1. 选择要下载的文件，点击页面里的 "生成链接" 来获取加速下载地址。
2. 使用鼠标右键点击链接，选择“导出到 ARIA2 RPC”，然后确定下载。

![网盘助手配合Aria2_for_Chrome](https://user-images.githubusercontent.com/7850715/73240648-81192f80-4197-11ea-8154-653c7b95adc1.gif)
> 图片出处:https://www.runningcheese.com/baiduyun

## 参考
- [Mac下配置Aria2来代替迅雷](https://www.damocles.me/2019/06/16/mac-aria2-configure/)
- [Aria2配置文件参数翻译详解](http://www.senra.me/aria2-conf-file-parameters-translation-and-explanation/)
- [关于aria2最完整的一篇](http://ivo-wang.github.io/2019/04/18/%E5%85%B3%E4%BA%8Earia2%E6%9C%80%E5%AE%8C%E6%95%B4%E7%9A%84%E4%B8%80%E7%AF%87/)
- [aria2c手册](https://aria2.github.io/manual/en/html/aria2c.html#aria2-conf)
