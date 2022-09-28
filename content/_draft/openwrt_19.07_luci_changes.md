+++
author = "FanHe"
title = "OpenWRT 19.07 Luci框架的改变"
date = 2021-04-21T08:47:25+08:00
description = ""
categories = [
    "OpenWRT开发"
]
+++

## 1. 简介

---

OpenWRT19.07的一个重大的改动是将之前OpenWRT的Luci框架做了比较大的调整，最主要集中在将Luci的渲染方式从之前的服务端渲染模式调整到客户端的渲染模式。据OpenWRT官方说这种改动可以提升默写老旧设备的性能，将渲染网页的工作从路由器转移到用户的客户端设备。

由于OpenWRT中luci-app非常众多，在本文写作时（最新版本是19.07.7）官方feeds中的luci-application仍然只改写了一小部分，后续估计官方会持续推进。鉴于目前19.07版本处于一种新旧方式过渡的阶段，大量老的使用lua编写的app尚未完全移植，因此如果发现老的app在19.07上运行异常（大部分都是由于cbi.lua造成的），官方给出了一些建议，包括：

- 安装luci-compat包（提供老代码的兼容方式运行）
- 如果页面加载缓慢，可以考虑安装 uhttpd-mod-ubus
- 页面加载缓慢或修改设置之后，建议重新打开浏览器标签页（或者重启浏览器）

以下我们对 19.07前后两种不同方式的luci-app开发作一个比较，挑选18.06（19.07的上一个稳定发行版）和19.07进行对比分析


## 2. 18.06的luci-app风格

### 2.1 luci-app开发的主要方式

主要是使用openwrt提供的框架，使用lua语言进行开发，采用MVC的架构方式，在 /usr/lib/lua/luci 目录中提供有 model、controller和view几个目录，在controller目录中通过编写lua文件，生成网页上的界面菜单并且指定如何处理点击菜单之后的响应。

响应主要通过3种方式提供：
（1）直接调用函数的方式，比如下面的示例展示了如何调用函数响应
```
module("luci.controller.myapp.mymodule", package.seeall)

function index()
    entry({"click", "here", "now"}, call("action_tryme"), "Click here", 10).dependent=false
end
 
function action_tryme()
    luci.http.prepare_content("text/plain")
    luci.http.write("Haha, rebooting now...")
    luci.sys.reboot()
end
```
  再我们点击某个菜单项时，这个菜单会调用action_tryme函数

  （2）通过调用一个html页面来响应

```
entry({"my", "new", "template"}, template("myapp-mymodule/helloworld"), "Hello world", 20).dependent=false
```
代码会调用 /usr/lib/lua/luci/view/myapp-mymodule/helloworld.htm这个页面来响应用户的点击

（3）通过CBI的方式来响应

这种方式也是用的比较多的一种方式，它通过编写一个lua文件来生成一个网页（包含大量的网页中控件），这些控件对应着lua中的一些类型，并且这些类型直接和uci的配置文件绑定，示例如下：

```
m = Map("network", "Network") -- 对应着一个配置文件 /etc/config/network

s = m:section(TypedSection, "interface", "Interfaces") -- s对应着network这个文件中的某一个配置块
s.addremove = true -- Allow the user to create and remove the interfaces
function s:filter(value)
   return value ~= "loopback" and value -- Don't touch loopback
end 
s:depends("proto", "static") -- Only show those with "static"
s:depends("proto", "dhcp") -- or "dhcp" as protocol and leave PPPoE and PPTP alone
...
gw = s:option(Value, "gateway", "Gateway")
gw:depends("proto", "static")
gw.rmempty = true -- Remove entry if it is empty

return m -- 返回配置
```
当用户进行操作之后，它会把用户的操作对应到uci文件中的具体选项中，在保存的时候写入到配置文件中


### 2.1 lua-app安装包的目录结构

老版本的luci-app的目录结构如下图所示：

```
/
├── etc/
│   ├── config/
│   │   └── shadowsocks                             // UCI 配置文件
│   │── init.d/
│   │   └── shadowsocks                             // init 脚本
│   └── uci-defaults/
│       └── luci-shadowsocks                        // uci-defaults 脚本
└── usr/
    ├── bin/
    │   └── ss-rules                                // 生成代理转发规则的脚本
    └── lib/
        └── lua/
            └── luci/                               // LuCI 部分
                ├── controller/
                │   └── shadowsocks.lua             // LuCI 菜单配置
                ├── i18n/                           // LuCI 语言文件目录
                │   └── shadowsocks.zh-cn.lmo
                └── model/
                    └── cbi/
                        └── shadowsocks/
                            ├── general.lua         // LuCI 基本设置
                            ├── servers.lua         // LuCI 服务器列表
                            ├── servers-details.lua // LuCI 服务器编辑
                            └── access-control.lua  // LuCI 访问控制
```
这是一个luci-app ipk包内的文件结构，这些文件会被拷贝到OpenWRT系统中对应的位置，可以看到主题的交互文件就是在/usr/lib/lua/luci的model、controller目录中。除此之外还需要搭配一些配置文件、应用程序的启动初始化脚本、翻译文件，构成整个应用程序。


## 3. 19.07的luci-app风格

新的luci-app把之前的模式进行了非常多的修改，首先一个最主要的改动就是减少了大量的lua代码，新的luci-app采用的是Javascript进行开发，并且页面基本上都是使用网页的方式来呈现（也就是直接编写html的文档，有点类似于18.06响应模式的第2种）

新的luci-app包的文件结构如下：

```
openwrt
┕feeds
  ┕luci
    ┕applications
      ┕luci-app-name #界面程序的主目录
        ┕htdocs
        ┊ ┕luci-static
        ┊   ┕resources
        ┊     ┕view
        ┊       ┕name.js # JavaScript 脚本界面文件。
        ┕po
        ┊ ┕zh_Hans # 此目录名称对应简体中文。
        ┊   ┕name.po # 界面语言翻译文件。
        ┕root
        ┊ ┕etc
        ┊ ┊ ┕uci-defaults
        ┊ ┊   ┕luci-app-name # 软件安装完毕后默认执行的脚本（一次性脚本），可选。
        ┊ ┕usr
        ┊   ┕share
        ┊     ┕luci
        ┊     ┊ ┕menu.d
        ┊     ┊   ┕luci-app-name.json # 界面菜单，在系统菜单中的名称、顺序等。
        ┊     ┕rpcd
        ┊       ┕acl.d
        ┊         ┕luci-app-name.json # 权限控制文件，管控界面能执行的各类操作。
        ┕Makefile # 编译文件。
```

下面这个链接给出了一个移植到新版本的luci-app程序相对于老版本的修改内容：

- luci-app-minidlna的改动

https://github.com/openwrt/luci/commit/9ae591b38fedf16c3e5c97350b7182c5e28ed71f#diff-27855472049b664538cca7ef50c43df8



## 4. 参考资料
---
[1.OpenWrt 19.07.0 - First Stable Release - 6 January 2020](https://openwrt.org/releases/19.07/notes-19.07.0)

[2. OpenWrt达人教程之开发人员入门指南](https://iyzm.net/openwrt/624.html)

[3. OpenWRT18.06 IPK的目录结构](https://github.com/shadowsocks/luci-app-shadowsocks/tree/v1.9.0)


