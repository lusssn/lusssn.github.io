---
title: Mac设置代理的方式
tags: Tools
date: 2019-01-11 19:42:23
---
在本地环境开发前端项目时，除了访问localhost外，还有通过自定义域名访问的需求，总结两种在Mac设置反向代理的方式。
<!-- more -->
<!-- toc -->
### 要设置代理的原因

折腾这些事情的背景需要说明一下。
首先在**测试环境**的域名为：`project.ndev.lusssn.cn`

在网页上登录成功后，后端会在cookie中存一个认证令牌，这个令牌在公司内部子系统中都是通用的，所以它的Domain为 `lusssn.cn`

我在**本地环境**开发通过 `localhost` 访问页面，借助Webpack反向代理，**调用测试环境的后台接口**（project.ndev.lusssn.cn）。然而

> localhost 不能访问Domain为 lusssn.cn 的cookie

### 解决方法

1. **将cookie的Domain更改为localhost**
  - 把Domain为 lusssn.cn 的cookie复制一份，Domain更改为localhost。
  - 这样的话，令牌过期一次就要手动复制粘贴一次。
2. **后端提供一个永久的令牌**
  - 感觉并不美啊，cookie清空了还得手动添加，备选方案。
3. **本地通过带 lusssn.cn 域的自定义域名访问**
  - 这样即不用复制粘贴，也不用手动添加，与测试环境共享cookie。
  - 堪称完美～

### 反向代理自定义域名

#### 1. Apache2代理

用Mac自带的Apache2起一个服务，设置反向代理，将自定义域名 `mydomain.lusssn.cn` 转发到localhost指定端口的服务即可。具体步骤和细节如下：

**Step 1**

终端运行 `sudo vi /etc/hosts`
host列表中添加 *127.0.0.1  mydomain.lusssn.cn*

**Step 2**

终端运行 `sudo open /etc/apache2/httpd.conf -a 'visual studio code' `
用vscode或其他文本编辑器打开apache2的配置文件。

通常配置中没有启用的配置会用井号注释掉，启用反向代理需要解注释以下内容：
- *Include /private/etc/apache2/extra/httpd-vhosts.conf*
- *LoadModule proxy_module modules/mod_proxy.so*
- *LoadModule proxy_http_module modules/mod_proxy_http.so*

找到ServerName配置，配置为 *ServerName localhost*

以上就完成了httpd.conf文件的修改。

**Step 3**

终端运行 `sudo open /etc/apache2/extra/httpd-vhosts.conf -a 'visual studio code' `
准备修改虚拟host文件的配置。

添加配置内容：
```
<VirtualHost *:80>
  ServerName mydomain.lusssn.cn
  ProxyRequests off
  <Proxy *>
    Order deny,allow
    Allow from all
  </Proxy>
  <Location />
    ProxyPass http://localhost:9026/
    ProxyPassReverse http://localhost:9026/
  </Location>
</VirtualHost>
```

一般此文件中原本有两个关于**dummy-host2. example. com**的虚拟host配置，如果你没有用到，就直接将相关配置注释掉。

**Step 4**

终端运行 `apachectl -t`
若结果为 **Syntax OK**，表示之前的配置都没有问题。若不是，报什么错就自己去搜吧。

接着继续运行 `sudo apachectl -k start` 启动apache服务。
浏览器访问 **mydomain.lusssn.cn** 按照配置自动跳转到localhost:9026服务器，就大功告成啦！

apachectl常用命令：
- 启动 `sudo apachectl -k start`
- 重启 `sudo apachectl -k restart`
- 停止 `sudo apachectl -k stop`
- 查看虚拟host列表 `apachectl -t -D DUMP_VHOSTS`

我自己配置的时候看了好些博客，没有一个是照做就能成功的，每一步骤总报错，不要怕，报错就去搜。

#### 2. Charles代理

用Apache2设置代理好在启动了守护进程之后，不关机就是无感的存在，但是配置过程有点复杂。
同事小兄弟就找到了借助Charles代理的方式，很简单。

**Step 1**

下载Charles，并安装好。具体过程就跳过了

**Step 2**

打开Charles，选择菜单中Tools > Map Remote选项，按需配置映射的域名，就完了。

![](tools.png) ![](edit-map.png)

**Tips**

配置完后，按道理打开浏览器，输入自定义的域名就能访问到本地页面了，但是有两个情况说明下：
**1. Chrome安装了代理插件，比如 Switchy Omega**
&emsp;&nbsp;这个情况下可能会导致浏览器内的网络访问先被插件拦截，走不到Charles。
&emsp;&nbsp;**解决办法：**关掉插件，或者插件中增设一个代理。
<span class="image-container">&emsp;&nbsp;![](omega.png)</span>

**2. Charles代理开启时，FireFox中访问网络可能会被安全机制拦截**
&emsp;&nbsp;**解决办法：**先将Charles的证书导出到本地
<span class="image-container">&emsp;&nbsp;![](save-certificate.png)</span>
&emsp;&nbsp;再导入FireFox中即可。
<span class="image-container">&emsp;&nbsp;![](import-certificate.png)</span>
