+++
author = "FanHe"
title = "OpenWRT-19.07Luci编辑菜单方法"
date = 2021-04-21T23:19:21+08:00
description = ""
categories = [
    ""
]
draft = false
+++

## 1. 简介
---

&emsp;OpenWRT从19.07开始逐步将网页的渲染模式从服务端移到客户端，由此带来的一个显著的变化是luci开发的Lua代码大幅减少，取而代之的是JavaScript代码的增加。今后在处理界面的逻辑上基本上都是使用JavaScript来处理了。OpenWRT 19.07系列应该是一个逐步转型的版本，在这个版本中可以支持两种模式的luci-app开发，包括：

- 使用传统的Lua方式编写网页界面（主要是 Call、Template、CBI这三种方式）
- 使用新式的JS+css+html的方式来编写界面

&emsp;在OpenWRT 19.07中由于有大量的app尚未迁移到新的模式，为了兼容老的luci-app，可以安装luci-app-compat这个工具包来实现运行老的luci-app

本文主要说明当前luci-app如何去编辑网页的菜单栏，把我们编写的程序放在对应的菜单栏下（菜单栏这个说法可能不准确，这个是我个人的称呼，指的是下图的内容）

![标题栏](/img/edit_menu_openwrt19.07/menubar.png)

&emsp;在本文写作时，最新的19.07版本是19.07.7，在安装这个版本后，我发现当前的luci-app主要有三种形态：

1. 完全没有迁移的app，还是使用18.06方式编写的界面
2. 部分迁移的app，使用兼容模式运行
3. 完全使用JavaScript改写的app

以一个对应的luci-app来说明每一种模式


## 2. 未迁移的luci-app
---

&emsp;在OpenWRT19.07.7的版本中，可以去opkg安装 luci-app-https-dns-proxy 这个luci-app，它就是尚未迁移的一个app，在安装之后，主要添加的文件包括：

- /usr/lib/lua/luci/controller/https-dns-proxy.lua

![cbi_controller](/img/edit_menu_openwrt19.07/style1_1.png)

这个在菜单栏上的Services目录下添加了 `DNS HTTPS Proxy`这一项，查看文档中的内容：

```
module("luci.controller.https-dns-proxy", package.seeall)
function index()
	if nixio.fs.access("/etc/config/https-dns-proxy") then
		entry({"admin", "services", "https-dns-proxy"}, cbi("https-dns-proxy"), _("DNS HTTPS Proxy")).acl_depends = { "luci-app-https-dns-proxy" }
		entry({"admin", "services", "https-dns-proxy", "action"}, call("https_dns_proxy_action"), nil).leaf = true
	end
end

function https_dns_proxy_action(name)
	local packageName = "https-dns-proxy"
	local http = require "luci.http"
	local sys = require "luci.sys"
	local util = require "luci.util"
	if name == "start" then
		sys.init.start(packageName)
	elseif name == "action" then
		util.exec("/etc/init.d/" .. packageName .. " reload >/dev/null 2>&1")
	elseif name == "stop" then
		sys.init.stop(packageName)
	elseif name == "enable" then
		sys.init.enable(packageName)
	elseif name == "disable" then
		sys.init.disable(packageName)
	end
	http.prepare_content("text/plain")
	http.write("0")
end
```
在传统的luci-app开发过程中，对于一个菜单的响应有3种方式：分别是执行指定方法（Action）、访问指定页面（Views）以及调用CBI Module。

```
第一种可以直接调用指定的函数，比如点击菜单项就直接重启路由器等等，比如写为“call("function_name")”，然后在lua文件下编写名为function_name的函数就可以调用了。
第二种可以访问指定的页面，比如写为“template("myapp/mymodule")”就可以调用/usr/lib/lua/luci/view/myapp/mymodule.htm文件了。
第三种方法无非是最方便的，比如写为“cbi("myapp/mymodule")”就可以调用/usr/lib/lua/luci/model/cbi/myapp/mymodule.lua文件了。
```

可以看到响应菜单的方式是通过调用cbi和call的方式进行的，cbi的model文件位置在 /usr/lib/lua/luci/model/cbi/https-dns-proxy.lua

![cbi_model](/img/edit_menu_openwrt19.07/style1_cbi_model.png)

以上就是传统的luci-app开发方式，主要使用lua语言进行操作的交互响应。


## 3. 部分迁移的luci-app
---

&emsp;部分迁移的luci-app主要是将菜单的响应部分迁移到 javascript中（/www/luci-static/resources)，在19.07.7下的 `luci-app-adblock` 就是一个部分迁移的例子

在 `luci-app-adblock` 中，配置菜单栏上的菜单项也是在controller目录中的adblock.lua文件中进行的，这个文件内容如下：

```
-- stub lua controller for 19.07 backward compatibility

module("luci.controller.adblock", package.seeall)

function index()
	entry({"admin", "services", "adblock"}, firstchild(), _("Adblock"), 60)
	entry({"admin", "services", "adblock", "overview"}, view("adblock/overview"), _("Overview"), 10)
	entry({"admin", "services", "adblock", "dnsreport"}, view("adblock/dnsreport"), _("DNS Report"), 20)
	entry({"admin", "services", "adblock", "blacklist"}, view("adblock/blacklist"), _("Edit Blacklist"), 30)
	entry({"admin", "services", "adblock", "whitelist"}, view("adblock/whitelist"), _("Edit Whitelist"), 40)
	entry({"admin", "services", "adblock", "logread"}, view("adblock/logread"), _("Log View"), 50)
end

```
可以看到它的调用方式不是传统luci-app方式那3种方式中的任何一种，而是一种全新的使用JavaScript进行响应的方式，这里面的view(adblock/*)对应的是/www/luci-static/resources/view 目录下的js文件

![cbi_model](/img/edit_menu_openwrt19.07/adblock.png)


也就是说在这种过渡方案模式下，有以下特点：

1. 菜单栏的配置还是使用传统的luci-app方式进行的，仍然是在 `controller` 目录中配置
2. 对于菜单栏的响应设置在新的JavaScript脚本中进行


## 4. 完全迁移的luci-app
---

上面提到了过渡模式下菜单栏和对菜单栏响应方式的变化，最新的OpenWRT的实现中，菜单栏和对菜单栏的响应都不在传统的 /usr/lib/lua/luci 目录下进行了，而是采用下面这种处理方式

- 菜单栏的配置修改到 /usr/share/luci/menu.d 目录中，并且配置文件使用.json文件
- 对菜单栏的响应修改到 /www/luci-static/resources 目录中，并且响应的脚本都是.js文件

我们查看这个menu.d目录中的 luci-base.json 文件，可以看到文件中列举出所有标题栏上显示的内容

```
{
	"admin": {
		"title": "Administration",
		"order": 10,
		"action": {
			"type": "firstchild",
			"recurse": true
		},
		"auth": {
			"methods": [ "cookie:sysauth" ],
			"login": true
		}
	},

	"admin/status": {
		"title": "Status",
		"order": 10,
		"action": {
			"type": "firstchild",
			"preferred": "overview",
			"recurse": true
		}
	},

	"admin/system": {
		"title": "System",
		"order": 20,
		"action": {
			"type": "firstchild",
			"preferred": "system",
			"recurse": true
		}
	},

	"admin/vpn": {
		"title": "VPN",
		"order": 30,
		"action": {
			"type": "firstchild",
			"recurse": true
		}
	},

	"admin/services": {
		"title": "Services",
		"order": 40,
		"action": {
			"type": "firstchild",
			"recurse": true
		}
	},
   ... //省略其他配置
}
```

&emsp;如果我们想要添加自己的顶层菜单，是可以直接编辑这个文件的。但是并不推荐这么做，因为如果每一个组织或个人开发的程序都要添加自己的顶层菜单，那么会造成这个文件修改的混乱，更好的办法是自己创建一个.json的文件，并采用类似luci-base.js 的写法，比如我想创建一个名称是"HZX"的顶层菜单，那么可以添加一个文件
luci-hzxtopmenu.js文件，文件内容如下：

```
{
	"admin/hzxtopmenu": {    //菜单对应在网页url中的地址后缀
		"title": "HZX",      //菜单栏上显示的名称
		"order": 80,         //菜单栏的显示顺序（越大越在后面）
		"action": {
			"type": "firstchild",
			"recurse": true
		}
	}
}

```
只添加这一个文件并不能在菜单栏上显示 "HZX" ，我们需要在 "HZX" 下面添加一个子菜单选项，添加方式也是模仿已有app的写法，比如我们创建一个luci-app-goshadowsock2.json的文件，文件内容如下：

```
{
	"admin/hzxtopmenu/goshadowsocks2": {
		"title": "GoShadowsocks2",
		"order": 10,
		"action": {
			"type": "view",
			"path": "goshadowsocks2/overview"
		}
	}
}
```
这样就可以在HZX菜单项的下面添加一个叫GoShadowsocks2的子菜单项，并且点击它之后的响应转到 /www/luci-static/resources/view/goshadowsocks2/overview.js 文件中去处理，后续要做的事情就是使用JavaScript脚本完善用户点击的响应。

下图是添加这些文件后的效果

![ss2](/img/edit_menu_openwrt19.07/ss2.png)


