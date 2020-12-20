## 8.1 真机远程代码调试

### 8.1.1 ios真机设备调试

1. 在手机是上开启网页检查器

设置-> safari 浏览器 -> 高级 -> 网页检测器

2. 开启Safari上的Develop功能

菜单 -> Preferences（偏好设置） -> Advanced（高级）

![](~@/engineering/devetoopios.png)

3. 使用数据线连接手机个开发主机

- 信任电脑

4. 打开devTool

Develop -> 你的ios设备 -> 要调试的页面

![](~@/engineering/connect.png)

### 8.1.2 Android真机设备调试

1. 在手机上开启USB调试功能

设置-> 开发者选项 -> USB调试

2. 使用数据线连接手机和开发主机

- 允许USB调试

3. 打开Chrome DevTools

- 在地址栏输入chrome://inspect
- 确保开启了 Discover USB devices

![](~@/engineering/andconnect.png)

## 8.2 无线调试工具-Weinre

- 无需数据线
- pc和移动都可以调试

1. 环境准备与安装

- 环境：任何node.js 环境
- 安装：`npm install -g weinre`

```bash
 weinre -h  #获取帮助信息
 weinre --boundHost=-all- --httpPort=1000 # 启动，boundHost为all是允许本机所有有效ip访问，默认端口8080
```
- 浏览打开：ip:1000 或 http://localhost:1000/

![](~@/engineering/weinre.png)

2. Target 页面配置

```html
<!-- 往被调试页面添加脚本 -->
<script src="http://ip:端口/target/target-script-min.js#anonymous"></script>
<!-- 示例 -->
<script src="http://172.20.10.11:1000/target/target-script-min.js#anonymous"></script>
```

3. 调试

调试移动手机的页面需要手机访问局域网内的页面服务（例如react开启了3000端口的服务，那么手机访问ip+3000）

![](~@/engineering/weinreremote.png)

扩展

[weinre官网](https://people.apache.org/~pmuellr/weinre/docs/latest/Home.html)

##  8.3 在移动端调试

主流的移动端devTool

- vConsole
- eruda

扩展资料

[vConsole项目主页](https://github.com/Tencent/vConsole)

[eruda项目主页](https://github.com/liriliri/eruda)

[创建一个vConsole 插件](https://github.com/Tencent/vConsole/blob/dev/doc/plugin_building_a_plugin.md)


##  8.4 使用代理服务器进行调试（charles）

### 8.4.1 常见的代理服务器

- Fiddler
  - C#编写
  - 正式版仅支持Windows
  - 请求展示：时间顺序
  - 支持解析HTTPS请求
  - 免费

- Charles
  - java编写
  - 多平台支持
  - 请求展示：树状结构
  - 不支持直接解析HTTPS请求
  -  付费获得更好体验

### 8.4.2 HTTP抓包

1. 移动端配置（配置代理）

点击手机连接的Wi-Fi->HTTP代理->配置代理

- 服务器：填入电脑的ip地址(可以在charles查看 help->Local IP Address)
- 端口：默认是8888

2. 开始抓包

工具栏第二个按钮（红色说明正在抓包）

![](~@/engineering/charles1.png)

### 8.4.3 HTTPS抓包

charles不能直接抓包https，所以我们还需要进行以下操作

1. 移动端配置代理

 与上面http配置完全相同

2. 添加要解析的域名列表

- 菜单栏Proxy-> SSL Proxying Settings
- 在Host一栏设置要解析的域名，也可以*表示所有的HTTPS都做解析,port:443；点击ok后重启charles。

![](~@/engineering/charles2.png)

<font color="red">**设置后重启charles**</font>

3. 信任Charles根证书

- 在移动端用浏览器访问`https:chls.pro/ssl`下载证书描述文件
  - 安卓：直接安装即可
  - IOS：1.在设置-通用-描述文件与设置管理中安装证书，2.然后开启 设置-通用-关于本机-针对根证书启用完全信任

🚀 HTTPS 协议

- https = http + sll（secure  sockets layer）

发送一个HTTPS请求的过程简单来说客户端是向服务端索取一个公钥放在客户端，公钥的载体就是数字证书；客户端通过公钥加密，服务端通过私钥解密。

🚀 charles 代理HTTPS请求的机制

charles做客户端和服务端的中介，代理了https请求；charles会动态为每个服务器生成由charles根证书签发的数字证书；当请求产生时客户端收到的不是
服务端的原始证书而是charles签发的证书，原始证书是保存在charles上的；客户端与服务端的通信是由charles公钥加密的，而charles与原始服务器的通信是
由服务端的原始证书加密的，charles通过自己的私钥解密，所以我们能通过charles看见未加密的数据。

🚀  可能出现的问题

1. 开启后电脑浏览器无法打开网页

关掉charles的External Proxy setting：点击顶部Proxy-> 去掉勾选 External Proxy Settings

2. 开启后手机浏览器无法打开网页

检测顶部Proxy-> macOS Proxy 是否勾选；需要勾选（window名称不一样）；

打开网络偏好设置-> 高级 -> 代理 -> 检测网页代理和安全网页代理是否开启（需要开启）

3. 关闭charles后 无法联网

打开网络偏好设置-> 高级 -> 代理 -> 检测网页代理和安全网页代理是否开启（不需要开启）

4. 抓取手机端的返回的是unknow

- 确保证书在手机端被信任

-  safari浏览器访问`https:chls.pro/ssl`下载证书描述文件时候，确保配置了http代理（ip:8888），
如果换电脑需要再次下载证书，可以在描述文件中查看证书，charles通用名称中会包含你电脑系统的名称

### 8.4.4 Map请求重定向

🚀 Map Local / Map Remote

- 生产环境代码打包压缩后不利于debug
- 调试时不方便频繁发布
- `Map Local`：将某请求重定向至本地某个文件
- `Map Remote`：将某请求重定向至另一个请求
- 将生成环境的代码替换成本地的代码

🚀 Map Remote

![](~@/engineering/mapremote.png)

尝试改动代码，查看页面变化

🚀 Map Local

![](~@/engineering/maplocal.png)

Map Local可以替换任意charles能抓包到的文件，甚至可以将api请求替换成本地的json文件，做接口的一个mock，
其实就是一个文件的代理功能。

如果想取消Map Local / Map Remote：charles顶部tools-> Map Local / Map Remote -> 不勾选enable map local/Remote-> ok

### 8.4.5 Rewrite 修改请求内容

🚀 Rewrite功能

- 可作用于：request & reponses
- 可修改：header、host、path、url、params、body等
- 采用新增、替换的形式

🚀 案例

向生产环境页面注入vConsole脚本

1. 顶部tools-> Rewrite 

添加匹配location
![](~@/engineering/rewrite1.png)

添加rewrite规则

![](~@/engineering/rewrite2.png)

将head标签替换成<script src="地址">new VConsole()</script></head>

## 8.5 在公网访问本地服务

### 8.5.1 内网穿透

#### 什么叫内网穿透

-  NAT（Network Address Translation）穿透 
- 从公网访问内网
- 在公网访问部署在本地服务器上的服务
- ngrok & localtunnel
  - 生成唯一可在公网访问的url，该url会在代理本地运行的web服务请求

### 8.5.2 localtunnel的使用

1. 安装

```bash
    npm install -g localtunnel
    lt --help # 帮助信息
```

2. 启动

```bash
    # -p 本地启动服务的端口，-s 指定自己子域名地址，也可以不指定
    lt -p 3000 -h https://tunnel.svrx.io -s mayi
    # 启动成功后 your url is: https://mayi.tunnel.svrx.io
    # 访问地址 https://mayi.tunnel.svrx.io
```
### 8.5.3 部署你的localtunnel服务

- localtunnel默认服务在外国，不稳定，访问速度慢
- 自己部署lt服务的机器需要满足：
  1. 支持DNS泛域名解析，比如：mydomain.com 和 *.mydomain.com
  2. Localtunnel服务端能监听任何非root权限的TCP端口

1. 配置dns解析
    添加两个A类记录 
    -  mydomain.com
    - *.mydomain.com

2. 启动localtunnel server

```bash
    git clone git://github.com/defunctzombie/localtunnel-server.git
    cd localtunnel-server
    npm install
    bin/server --port 1234
    bin/server --port 1234 --domain sub.mydomain.com
```
3. 在localtunnel客户端使用部署的host

```bash
    lt --host http://sub.mydomain.com:1234  --port 8000
```

