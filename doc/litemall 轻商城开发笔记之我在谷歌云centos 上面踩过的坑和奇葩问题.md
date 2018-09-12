

# litemall 轻商城开发笔记之我在谷歌云 centos 上面踩过的坑和奇葩问题

首先介绍下项目的背景，litemall 项目是码云代码托管平台开源软件页面推荐的项目，无意之间看到的，逛了一下他的演示站，我立刻被震撼到了，看过许多开源作品，只有 litemall 的 web 后台管理系统让我眼前一亮，VUE 框架开发的 WEB 页面果然十分的优雅和美观，于是出于职业习惯和强烈的研究欲望，我马上决定把这个项目运行在我那闲置多年的谷歌云服务器上。先看下效果图：

[![](https://camo.githubusercontent.com/35e38a0c057a56de4ec2018e68417786eb00f04d/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383031323332382d6368726f6d6532303138303830315f3233323831392e706e672d7379)](https://camo.githubusercontent.com/35e38a0c057a56de4ec2018e68417786eb00f04d/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383031323332382d6368726f6d6532303138303830315f3233323831392e706e672d7379)

[![](https://camo.githubusercontent.com/3b9a7dc86faadaa15d192eb63336b4cc318e5c6c/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383031323332392d6368726f6d6532303138303830315f3233323934352e706e672d7379)](https://camo.githubusercontent.com/3b9a7dc86faadaa15d192eb63336b4cc318e5c6c/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383031323332392d6368726f6d6532303138303830315f3233323934352e706e672d7379)

[![](https://camo.githubusercontent.com/435fc3097d3928de52b279e336f1468a0cddb6fa/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383031323333312d6368726f6d6532303138303830315f3233333132392e706e672d7379)](https://camo.githubusercontent.com/435fc3097d3928de52b279e336f1468a0cddb6fa/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383031323333312d6368726f6d6532303138303830315f3233333132392e706e672d7379)

[![](https://camo.githubusercontent.com/ce2aa72191ac06f0190633e60a2f766e51fba672/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383031323333322d6368726f6d6532303138303830315f3233333234362e706e672d7379)](https://camo.githubusercontent.com/ce2aa72191ac06f0190633e60a2f766e51fba672/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383031323333322d6368726f6d6532303138303830315f3233333234362e706e672d7379)

1 - 关于一年免费试用谷歌云搭建科学上网服务器的文章我推荐这个，写得非常详细：[https://www.wmsoho.com/google-cloud-platform-ssr-bbr-tutorial/](https://www.wmsoho.com/google-cloud-platform-ssr-bbr-tutorial/)

原版 & 魔改版 Google BBR 拥塞控制算法一键安装脚本：[https://www.vultrcn.com/5.html](https://www.vultrcn.com/5.html)

![image](https://user-images.githubusercontent.com/12899262/45407978-91f9fb80-b69d-11e8-9a22-0d9bcc0ed2ed.png)


2 - 有了服务器和科学上网环境之后，新建虚拟机和配置网络防火墙之类的此处略过一万字。本文以谷歌云 centos 6 为例，其他系统仅供参考。提一下谷歌云默认的云虚拟机无法 root 远程登录，ssh 密钥登录方式见推荐文章。可以先通过网站的内置控制台进入云虚拟机更改以下代码，即可实现远程 root 登录：

2.1 - 以非 root 用户进入控制台后，

`sudo su - root`

进入 root 账户

`passwd root`

设置 root 账户的密码

然后修改 SSH 配置文件

```
vim /etc/ssh/sshd_config
```

找到 PermitRootLogin 和 PasswordAuthentication，参考如下设置：

```
PermitRootLogin yes //默认为no，需要开启root用户访问改为yes
PasswordAuthentication yes //默认为no，改为yes开启密码登陆
```

重启 SSH 服务使修改生效

`/etc/init.d/ssh restart`

3 - 接下来，快速部署 LNM（Linux+Nginx+Mysql）环境。

3.1 - 本文使用远程终端神器 mobaxterm 来管理服务器，至于这个软件有多强大，看图：

[![](https://camo.githubusercontent.com/82b35966b1c488dbda3fec9c976104d00395846a/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303431342d32303138303830325f3034313431362e706e672d7379)](https://camo.githubusercontent.com/82b35966b1c488dbda3fec9c976104d00395846a/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303431342d32303138303830325f3034313431362e706e672d7379)

登入新建的云虚拟机：

[![](https://camo.githubusercontent.com/c7ec74244b78049fdf09a6175bd1d70fb05ad936/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383031323333352d4d6f6261587465726d32303138303830315f3233333533332e706e672d7379)](https://camo.githubusercontent.com/c7ec74244b78049fdf09a6175bd1d70fb05ad936/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383031323333352d4d6f6261587465726d32303138303830315f3233333533332e706e672d7379)

3.2 - 打开 oneinstack 网站：[https://oneinstack.com/auto/](https://oneinstack.com/auto/)，使用一键部署命令，只需要 20 分钟左右就可以快速完成 LNM 环境部署。

[![](https://camo.githubusercontent.com/6871d93f089fe4514b912ba809a7d40a6a48c2b9/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383031323334322d6368726f6d6532303138303830315f3233343230352e706e672d7379)](https://camo.githubusercontent.com/6871d93f089fe4514b912ba809a7d40a6a48c2b9/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383031323334322d6368726f6d6532303138303830315f3233343230352e706e672d7379)

我连代码都给你准备好了（一行），redis 可选，你自己决定，如果不装 tomcat ，你需要自己安装 jdk 1.8：

```
wget http://mirrors.linuxeye.com/oneinstack-full.tar.gz && tar xzf
oneinstack-full.tar.gz && ./oneinstack/install.sh --nginx_option 1 --db_option 2
--dbinstallmethod 1 --dbrootpwd oneinstack --redis --iptables --reboot
```

重启之后，mysql 和 nginx 已经默认启动，此时访问你的云端虚拟主机 ip 应该可以访问到 nginx 页面，> 注意 1：如果无法访问 80 端口，请依次检查 --- 云主机防火墙 --- 云平台防火墙 --- 端口占用 ---- 实在不行重启。

4 - 准备项目代码，修改配置文件并打包。

4.2 - 下载 litemall 开源项目的代码到本地，码云：[https://gitee.com/jkqn/litemall/blob/v0.8.0](https://gitee.com/jkqn/litemall/blob/v0.8.0)

推荐下载标签版本，如果 git 克隆到本地则是下载默认主分支，至于 idea 切换到标签版本我不太熟。也可以使用 IDEA 编辑器 git 插件克隆，> 注意 2：克隆后导入方式选择 maven，否则项目无法编译。

[![](https://camo.githubusercontent.com/9daf8a3a207020e5fba500f4c2617b9f9b7b27b3/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303431392d32303138303830325f3034313934372e706e672d7379)](https://camo.githubusercontent.com/9daf8a3a207020e5fba500f4c2617b9f9b7b27b3/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303431392d32303138303830325f3034313934372e706e672d7379)

[![](https://camo.githubusercontent.com/79db05bdd4f547502b373ff3c305e0fa7204e022/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303035382d32303138303830325f3030353830302e706e672d7379)](https://camo.githubusercontent.com/79db05bdd4f547502b373ff3c305e0fa7204e022/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303035382d32303138303830325f3030353830302e706e672d7379)

导入后效果如下：

[![](https://camo.githubusercontent.com/01daa3e3316505e5636bbe420ebc6723e6b297fb/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303030362d69646561363432303138303830325f3030303632332e706e672d7379)](https://camo.githubusercontent.com/01daa3e3316505e5636bbe420ebc6723e6b297fb/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303030362d69646561363432303138303830325f3030303632332e706e672d7379)

[![](https://camo.githubusercontent.com/374a574da9c475573472f401481536673db8fc3e/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303031352d6368726f6d6532303138303830325f3030313531312e706e67)](https://camo.githubusercontent.com/374a574da9c475573472f401481536673db8fc3e/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303031352d6368726f6d6532303138303830325f3030313531312e706e67)

[![](https://camo.githubusercontent.com/9899537181485717b7c19cf8649371775eefd5c4/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303031362d6368726f6d6532303138303830325f3030313631372e706e67)](https://camo.githubusercontent.com/9899537181485717b7c19cf8649371775eefd5c4/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303031362d6368726f6d6532303138303830325f3030313631372e706e67)

当前开发阶段的方案：

*   MySQL 数据访问地址 jdbc:mysql://localhost:3306/litemall
*   litemall-wx-api 后台服务地址 [http://localhost:8080/wx，数据则来自 MySQL](http://localhost:8080/wx%EF%BC%8C%E6%95%B0%E6%8D%AE%E5%88%99%E6%9D%A5%E8%87%AAMySQL)
*   litemall-admin-api 后台服务地址 [http://localhost:8080/admin, 数据则来自 MySQL](http://localhost:8080/admin,%E6%95%B0%E6%8D%AE%E5%88%99%E6%9D%A5%E8%87%AAMySQL)
*   litemall-admin 前端访问地址 [http://localhost:9527](http://localhost:9527), 数据来自 litemall-admin-api

litemall-wx 没有前端访问地址，而是直接在微信小程序工具上编译测试开发，最终会部署到微信官方平台（即不需要自己部署 web 服务器），而数据则来自 litemall-wx-api。

部署阶段可参考以上配置修改相应地址。本地开发测试过程如下：

4.2 - 数据库环境设置过程如下：

本机安装 MySQL，创建数据库、用户权限、数据库表和测试数据。

数据库文件在 litemall-db/sql 文件夹中，其中 litemall_schema.sql 创建数据库和用户权限， litemall_table.sql 则创建表，litemall_data.sql 则是测试数据。

> 注意 3：如果数据库连接异常，本地 localhost 和 127.0.0.1 不相同，必须一一对应。

[![](https://camo.githubusercontent.com/c968a87b753cdccd384c644d777504aad89e7328/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303432352d32303138303830325f3034323531342e706e672d7379)](https://camo.githubusercontent.com/c968a87b753cdccd384c644d777504aad89e7328/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303432352d32303138303830325f3034323531342e706e672d7379)

![image](https://user-images.githubusercontent.com/12899262/45408056-d71e2d80-b69d-11e8-9ef2-e9a7b4f609d3.png)

4.3 - 接下来先 IDEA 双击执行 maven 菜单下的 litemall 根节点的 clean，再双击 install 安装依赖库。

4.4-IDEA 编辑器在 litemall-all 模块的 Application 类 右键 Run Application.main() 方式运行该模块，运行时会自动编译再运行。启动成功后可以看到，litemall-wx-api 等模块多了 target 文件夹，里面是编译出的文件。打开浏览器，输入

```
http://localhost:8080/wx/index/index
http://localhost:8080/admin/index/index
```

如果出现 JSON 数据，则 litemall-all 模块运行正常。

4.5 - 微信小程序开发测试：

安装微信小程序开发工具，导入本项目的 litemall-wx 模块文件夹，编译前，请确定 litemall-all 模块已经运行，litemall-wx 模块的 config 文件夹中的 api.js 已经设置正确的后台数据服务地址；点击编译，如果出现数据和图片，则运行正常。详情查看项目文档。

[![](https://camo.githubusercontent.com/ef037081f4821c3ad5ffb99135fb3c84fe3c3609/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303433302d32303138303830325f3034333031332e706e672d7379)](https://camo.githubusercontent.com/ef037081f4821c3ad5ffb99135fb3c84fe3c3609/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303433302d32303138303830325f3034333031332e706e672d7379)

4.6-Vue 开发环境部署：

安装 nodejs，安装依赖库：

```
cd litemall/litemall-admin
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

> 注意 4：为了防止 cnpm 产生的文件太多导致 IDEA 索引卡死，强烈建议先在 litemall-admin 下面手动新建 node_modules 文件夹，并将其排除，避免 idea 索引这个目录导致卡死。

[![](https://camo.githubusercontent.com/fa3742e78415dcd2c3490b07a245c6152aacf6f3/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303433312d32303138303830325f3034333132362e706e672d7379)](https://camo.githubusercontent.com/fa3742e78415dcd2c3490b07a245c6152aacf6f3/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303433312d32303138303830325f3034333132362e706e672d7379)

接下来：

```
cd litemall/litemall-admin
 cnpm install
```

编译并运行 web 项目

`cnpm run dev`

然后，打开浏览器，输入 [http://localhost:9527。](http://localhost:9527%E3%80%82) 如果出现管理后台登录页面，则表明管理后台的前端运行正常；

请确定 litemall-all 模块已经运行，然后点击登录，如果能够成功登录，则表明管理后台的前端和后端对接成功，运行正常。

5 - 项目打包，整理部署文件并上传云主机。

5.1 - 在主机打包项目到 deploy 目录：

5.1.1 - 复制 sql 文件到 deploy/db 目录下：itemall_schema.sql、litemall_table.sql、litemall_data.sql；

5.1.2 - 编译打包部署环境下的 web 文件 dist，并将 litemall-admin/dist 目录复制到 deploy 下面：

```
cd ./litemall-admin
cnpm install #已执行过可省略
cnpm run build:dep
```

5.1.3 - 执行 mvn 菜单的 litemall 根节点 clean 和 package，打包，然后把生成的可执行 jar 包 litemall-all/target/litemall-all-*-exec.jar 复制到 / deploy/litemall/litemall.jar；

5.1.4 - 修改 deploy/litemall 文件夹下面的 *.yml 外部配置文件，当 litemall-all 模块启动时会 加载外部配置文件，而覆盖默认 jar 包内部的配置文件。 例如，配置文件中一些地方需要设置成远程主机的 IP 地址。

> 注意 5：要使用绝对地址，而不要使用相对地址比如 localhost。

5.1.5-deploy 文件夹结构包含 dist 网页静态文件，db 数据库文件，litemall-all 主程序文件，然后将 deploy 上传到云主机根目录，推荐使用 mobaxterm 直接拖拽；

6 - 项目部署：修改云主机配置文件并启动项目，本文以 centos 6 为例。

6.1 - 确保远程主机环境 MySQL 和 JDK1.8 已经安装好，确保云主机平台的全局防火墙和云主机防火墙 iptables 已经允许相应的端口 3306 80 8080 等。安装过程参考 3.2。

6.2 - 依次按顺序导入 db/litemall_schema.sql litemall_table.sql litemall_data.sql，顺序不可打乱。

6.3 - 启动 springboot 服务：

```
chmod a+x /deploy/litemall/litemall.jar     #增加全部用户可执行权限`
sudo ln -f -s /deploy/litemall/litemall.jar /etc/init.d/litemall    #创建软连接注册服务
sudo service litemall start
```

> 注意 6：如果提示 unable to find java ，而且 java -version 显示正确的话，可能的原因是 jdk 安装方式不规范，需要建立 java 命令的软链接到 / sbin。

如：`ln -s /usr/local/jdk/bin/java /sbin/java`

[![](https://camo.githubusercontent.com/95fda2efc58af62685cc6d17887f5cfa3bf3e99e/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303433322d32303138303830325f3034333232342e706e672d7379)](https://camo.githubusercontent.com/95fda2efc58af62685cc6d17887f5cfa3bf3e99e/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303433322d32303138303830325f3034333232342e706e672d7379)

参考来源：[https://www.jianshu.com/p/563497a6e1a7](https://www.jianshu.com/p/563497a6e1a7)

> 注意：7 如果还是提示 failed to start ，

[![](https://camo.githubusercontent.com/e760f986f9eaa8ec084f7a20d836b0b4e500487e/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303433322d32303138303830325f3034333235352e706e672d7379)](https://camo.githubusercontent.com/e760f986f9eaa8ec084f7a20d836b0b4e500487e/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303433322d32303138303830325f3034333235352e706e672d7379)

那就该祭出终极大杀器了，这个问题我找遍了 google 才在一个国外网站找到答案：

[![](https://camo.githubusercontent.com/8727d26370bd6a3cfd007767cac7757e94001f16/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303433332d32303138303830325f3034333332322e706e672d7379)](https://camo.githubusercontent.com/8727d26370bd6a3cfd007767cac7757e94001f16/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303433332d32303138303830325f3034333332322e706e672d7379)

下面先说一下我是怎么解决这个问题的，因为尝试了所有的办法去启动服务失败了，我就去查看了 litemall 服务启动日志 / var/logs/litemall.log，我在文件中发现了如下字样：

"start-stop-daemon: unrecognized option'--no-close'" 我用 google 搜了一下，找到 stackoverflow 社区的一个帖子，跟我的案例很相似。案后翻译中文，按照提示执行最关键的一句：

Create a config file /var/appname/appname.conf with the following content

`USE_START_STOP_DAEMON=false`

果然，程序正确启动了，我激动得流下了感动的眼泪。现在，我们的项目核心 jar 包已经运行，由于已经集成 tomcat，所以无需再部署 tomcat，浏览器访问如下地址，测试能否正常显示。

```
http://xxx.xxx.xxx.xxx:8080/wx/index/index
http://xxx.xxx.xxx.xxx:8080/admin/index/index
http://xxx.xxx.xxx.xxx:8080/\#/login
```

7 - 修改 nginx 并配置域名，ssl 证书可选。

在这个项目的 nginx 配置过程，我遭遇了无尽的挫折，明明平时用的很正常的 nginx，偏偏这次无法访问 80 端口，

> 后来走投无路的我干脆重启云主机【注意 8】，万万没想到，80 端口竟然开放了。

然后修改 nginx.conf 文件反向代理相应的服务，终于可以通过 [java666.top](/geekyouth/litemall-kl/blob/master/doc/java666.top) 来访问项目演示啦，此处省略 500 字。至于 ssl 证书配置，请参考文档。

8 - 测试效果：

[![](https://camo.githubusercontent.com/af1b3925e56757b999564d33114f5aaf4cc9cb41/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303335352d6368726f6d6532303138303830325f3033353532372e706e672d7379)](https://camo.githubusercontent.com/af1b3925e56757b999564d33114f5aaf4cc9cb41/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303335352d6368726f6d6532303138303830325f3033353532372e706e672d7379)

[![](https://camo.githubusercontent.com/fb215d6e4212957ea05ee840912b42430c427c52/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303335362d6368726f6d6532303138303830325f3033353633392e706e672d7379)](https://camo.githubusercontent.com/fb215d6e4212957ea05ee840912b42430c427c52/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303335362d6368726f6d6532303138303830325f3033353633392e706e672d7379)

[![](https://camo.githubusercontent.com/e66c8c16f7e3a194a7341339c0328266f3d6bc2c/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303335372d6368726f6d6532303138303830325f3033353733302e706e672d7379)](https://camo.githubusercontent.com/e66c8c16f7e3a194a7341339c0328266f3d6bc2c/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303335372d6368726f6d6532303138303830325f3033353733302e706e672d7379)

[![](https://camo.githubusercontent.com/a37d587c94eae8d2dcc82732d117327249639965/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303335382d6368726f6d6532303138303830325f3033353830392e706e672d7379)](https://camo.githubusercontent.com/a37d587c94eae8d2dcc82732d117327249639965/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303335382d6368726f6d6532303138303830325f3033353830392e706e672d7379)

[![](https://camo.githubusercontent.com/98cd0c27591e7664dc3a9f91d8cd04d2e3967881/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303335392d6368726f6d6532303138303830325f3033353930392e706e672d7379)](https://camo.githubusercontent.com/98cd0c27591e7664dc3a9f91d8cd04d2e3967881/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303335392d6368726f6d6532303138303830325f3033353930392e706e672d7379)

![image](https://user-images.githubusercontent.com/12899262/45408118-003ebe00-b69e-11e8-99ba-524e14f58485.png)

[![](https://camo.githubusercontent.com/6a75cc2f0a1ff2de902b3dd76b916c92075ffffa/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303430302d6368726f6d6532303138303830325f3034303031382e706e672d7379)](https://camo.githubusercontent.com/6a75cc2f0a1ff2de902b3dd76b916c92075ffffa/687474703a2f2f7037666372713265342e626b742e636c6f7564646e2e636f6d2f3230313831383032303430302d6368726f6d6532303138303830325f3034303031382e706e672d7379)

想要快速查找本文的注意事项，请使用 ctrl + f 搜索关键字 “注意”。。。。

<sr-plugin-count>共计：5418 个字</sr-plugin-count>
