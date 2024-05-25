# Update-the-IPset-of-MWAN3-helper

这个项目原本是为了定期更新mwan3 helper中的IP段的，但是效果并不好，因此采用别的方式使用IPset分流。无需安装mwan3 helper，通过终端添加ipset并自动更新，设置持久化和开机自启。在同一个IPset内，不能同时调用IPv4和IPv6，因此选择分开执行。mwan3 helper内的IP段从未更新且仅有IPv4地址，对于多宽带分流来说非常鸡肋。

使用时请确保安装了`ipset`、`curl`、`xargs`依赖

# 终端内首次运行

## IPv6 IPset

```
ipset_name="NAME6"; url="URL"; mkdir -p /etc/ipset_configs; echo $url > /etc/ipset_configs/ipset_$ipset_name.url; ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet6; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nUSE_PROCD=1\nstart_service() {\n    [ -f /etc/ipset_configs/ipset_$ipset_name.save ] && ipset restore < /etc/ipset_configs/ipset_$ipset_name.save || { ipset create '"$ipset_name"' hash:net family inet6; curl -s $(cat /etc/ipset_configs/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}; }\n}\nstop_service() {\n    ipset save $ipset_name > /etc/ipset_configs/ipset_$ipset_name.save\n    ipset destroy $ipset_name\n}\nservice_triggers() {\n    procd_add_reload_trigger ipset_$ipset_name\n}\n' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; }; curl -s $(cat /etc/ipset_configs/ipset_$ipset_name.url) > /etc/ipset_configs/ipset_$ipset_name.txt
```

## IPv4 IPset

```
ipset_name="NAME4"; url="URL"; mkdir -p /etc/ipset_configs; echo $url > /etc/ipset_configs/ipset_$ipset_name.url; ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nUSE_PROCD=1\nstart_service() {\n    [ -f /etc/ipset_configs/ipset_$ipset_name.save ] && ipset restore < /etc/ipset_configs/ipset_$ipset_name.save || { ipset create '"$ipset_name"' hash:net family inet; curl -s $(cat /etc/ipset_configs/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}; }\n}\nstop_service() {\n    ipset save $ipset_name > /etc/ipset_configs/ipset_$ipset_name.save\n    ipset destroy $ipset_name\n}\nservice_triggers() {\n    procd_add_reload_trigger ipset_$ipset_name\n}\n' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; }; curl -s $(cat /etc/ipset_configs/ipset_$ipset_name.url) > /etc/ipset_configs/ipset_$ipset_name.txt
```

> 将`NAME4`、`NAME6`、`URL`自定义，注意：`IPv6 IPset`和`IPv4 IPset`并不一致，区别在于`inet6`和`inet`。IP段被缓存在/etc/ipset_configs的txt文件之中

# 定时更新

后续只需要运行以下两行命令就可以更新IPset

```
ipset_name="NAME"; mkdir -p /etc/ipset_configs; ipset flush $ipset_name; url=$(cat /etc/ipset_configs/ipset_$ipset_name.url); curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; curl -s $url > /etc/ipset_configs/ipset_$ipset_name.txt
```

> 只需要修改`NAME`即可，IPv4和IPv6的后续命令是通用的

在命令前面加入（* * * * * ），就可以在计划任务中定期运行，注意空格位置

比如

```
0 20 * * * ipset_name="cn6"; mkdir -p /etc/ipset_configs; ipset flush $ipset_name; url=$(cat /etc/ipset_configs/ipset_$ipset_name.url); curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; curl -s $url > /etc/ipset_configs/ipset_$ipset_name.txt
```

意味着每天`20`点自动更新`cn6`的IP段

# 命令

## cn6

```
ipset_name="cn6"; url="https://raw.githubusercontent.com/mayaxcn/china-ip-list/master/chnroute_v6.txt"; mkdir -p /etc/ipset_configs; echo $url > /etc/ipset_configs/ipset_$ipset_name.url; ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet6; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nUSE_PROCD=1\nstart_service() {\n    [ -f /etc/ipset_configs/ipset_$ipset_name.save ] && ipset restore < /etc/ipset_configs/ipset_$ipset_name.save || { ipset create '"$ipset_name"' hash:net family inet6; curl -s $(cat /etc/ipset_configs/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}; }\n}\nstop_service() {\n    ipset save $ipset_name > /etc/ipset_configs/ipset_$ipset_name.save\n    ipset destroy $ipset_name\n}\nservice_triggers() {\n    procd_add_reload_trigger ipset_$ipset_name\n}\n' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; }; curl -s $(cat /etc/ipset_configs/ipset_$ipset_name.url) > /etc/ipset_configs/ipset_$ipset_name.txt
```

## cmcc6

```
ipset_name="cmcc6"; url="https://gaoyifan.github.io/china-operator-ip/cmcc6.txt"; mkdir -p /etc/ipset_configs; echo $url > /etc/ipset_configs/ipset_$ipset_name.url; ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet6; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nUSE_PROCD=1\nstart_service() {\n    [ -f /etc/ipset_configs/ipset_$ipset_name.save ] && ipset restore < /etc/ipset_configs/ipset_$ipset_name.save || { ipset create '"$ipset_name"' hash:net family inet6; curl -s $(cat /etc/ipset_configs/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}; }\n}\nstop_service() {\n    ipset save $ipset_name > /etc/ipset_configs/ipset_$ipset_name.save\n    ipset destroy $ipset_name\n}\nservice_triggers() {\n    procd_add_reload_trigger ipset_$ipset_name\n}\n' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; }; curl -s $(cat /etc/ipset_configs/ipset_$ipset_name.url) > /etc/ipset_configs/ipset_$ipset_name.txt
```

## cnc6

```
ipset_name="cnc6"; url="https://gaoyifan.github.io/china-operator-ip/unicom6.txt"; mkdir -p /etc/ipset_configs; echo $url > /etc/ipset_configs/ipset_$ipset_name.url; ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet6; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nUSE_PROCD=1\nstart_service() {\n    [ -f /etc/ipset_configs/ipset_$ipset_name.save ] && ipset restore < /etc/ipset_configs/ipset_$ipset_name.save || { ipset create '"$ipset_name"' hash:net family inet6; curl -s $(cat /etc/ipset_configs/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}; }\n}\nstop_service() {\n    ipset save $ipset_name > /etc/ipset_configs/ipset_$ipset_name.save\n    ipset destroy $ipset_name\n}\nservice_triggers() {\n    procd_add_reload_trigger ipset_$ipset_name\n}\n' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; }; curl -s $(cat /etc/ipset_configs/ipset_$ipset_name.url) > /etc/ipset_configs/ipset_$ipset_name.txt
```

## ct6

```
ipset_name="ct6"; url="https://gaoyifan.github.io/china-operator-ip/chinanet6.txt"; mkdir -p /etc/ipset_configs; echo $url > /etc/ipset_configs/ipset_$ipset_name.url; ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet6; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nUSE_PROCD=1\nstart_service() {\n    [ -f /etc/ipset_configs/ipset_$ipset_name.save ] && ipset restore < /etc/ipset_configs/ipset_$ipset_name.save || { ipset create '"$ipset_name"' hash:net family inet6; curl -s $(cat /etc/ipset_configs/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}; }\n}\nstop_service() {\n    ipset save $ipset_name > /etc/ipset_configs/ipset_$ipset_name.save\n    ipset destroy $ipset_name\n}\nservice_triggers() {\n    procd_add_reload_trigger ipset_$ipset_name\n}\n' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; }; curl -s $(cat /etc/ipset_configs/ipset_$ipset_name.url) > /etc/ipset_configs/ipset_$ipset_name.txt
```

## cn4

```
ipset_name="cn4"; url="https://raw.githubusercontent.com/mayaxcn/china-ip-list/master/chnroute.txt"; mkdir -p /etc/ipset_configs; echo $url > /etc/ipset_configs/ipset_$ipset_name.url; ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nUSE_PROCD=1\nstart_service() {\n    [ -f /etc/ipset_configs/ipset_$ipset_name.save ] && ipset restore < /etc/ipset_configs/ipset_$ipset_name.save || { ipset create '"$ipset_name"' hash:net family inet; curl -s $(cat /etc/ipset_configs/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}; }\n}\nstop_service() {\n    ipset save $ipset_name > /etc/ipset_configs/ipset_$ipset_name.save\n    ipset destroy $ipset_name\n}\nservice_triggers() {\n    procd_add_reload_trigger ipset_$ipset_name\n}\n' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; }; curl -s $(cat /etc/ipset_configs/ipset_$ipset_name.url) > /etc/ipset_configs/ipset_$ipset_name.txt
```

## cmcc4

```
ipset_name="cmcc4"; url="https://gaoyifan.github.io/china-operator-ip/cmcc.txt"; mkdir -p /etc/ipset_configs; echo $url > /etc/ipset_configs/ipset_$ipset_name.url; ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nUSE_PROCD=1\nstart_service() {\n    [ -f /etc/ipset_configs/ipset_$ipset_name.save ] && ipset restore < /etc/ipset_configs/ipset_$ipset_name.save || { ipset create '"$ipset_name"' hash:net family inet; curl -s $(cat /etc/ipset_configs/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}; }\n}\nstop_service() {\n    ipset save $ipset_name > /etc/ipset_configs/ipset_$ipset_name.save\n    ipset destroy $ipset_name\n}\nservice_triggers() {\n    procd_add_reload_trigger ipset_$ipset_name\n}\n' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; }; curl -s $(cat /etc/ipset_configs/ipset_$ipset_name.url) > /etc/ipset_configs/ipset_$ipset_name.txt
```

## cnc4

```
ipset_name="cnc4"; url="https://gaoyifan.github.io/china-operator-ip/unicom.txt"; mkdir -p /etc/ipset_configs; echo $url > /etc/ipset_configs/ipset_$ipset_name.url; ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nUSE_PROCD=1\nstart_service() {\n    [ -f /etc/ipset_configs/ipset_$ipset_name.save ] && ipset restore < /etc/ipset_configs/ipset_$ipset_name.save || { ipset create '"$ipset_name"' hash:net family inet; curl -s $(cat /etc/ipset_configs/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}; }\n}\nstop_service() {\n    ipset save $ipset_name > /etc/ipset_configs/ipset_$ipset_name.save\n    ipset destroy $ipset_name\n}\nservice_triggers() {\n    procd_add_reload_trigger ipset_$ipset_name\n}\n' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; }; curl -s $(cat /etc/ipset_configs/ipset_$ipset_name.url) > /etc/ipset_configs/ipset_$ipset_name.txt
```

## ct4

```
ipset_name="ct4"; url="https://gaoyifan.github.io/china-operator-ip/chinanet.txt"; mkdir -p /etc/ipset_configs; echo $url > /etc/ipset_configs/ipset_$ipset_name.url; ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nUSE_PROCD=1\nstart_service() {\n    [ -f /etc/ipset_configs/ipset_$ipset_name.save ] && ipset restore < /etc/ipset_configs/ipset_$ipset_name.save || { ipset create '"$ipset_name"' hash:net family inet; curl -s $(cat /etc/ipset_configs/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}; }\n}\nstop_service() {\n    ipset save $ipset_name > /etc/ipset_configs/ipset_$ipset_name.save\n    ipset destroy $ipset_name\n}\nservice_triggers() {\n    procd_add_reload_trigger ipset_$ipset_name\n}\n' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; }; curl -s $(cat /etc/ipset_configs/ipset_$ipset_name.url) > /etc/ipset_configs/ipset_$ipset_name.txt
```

# 计划任务

```
0 15 * * * ipset_name="cmcc6"; mkdir -p /etc/ipset_configs; ipset flush $ipset_name; url=$(cat /etc/ipset_configs/ipset_$ipset_name.url); curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; curl -s $url > /etc/ipset_configs/ipset_$ipset_name.txt
0 16 * * * ipset_name="cnc6"; mkdir -p /etc/ipset_configs; ipset flush $ipset_name; url=$(cat /etc/ipset_configs/ipset_$ipset_name.url); curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; curl -s $url > /etc/ipset_configs/ipset_$ipset_name.txt
0 17 * * * ipset_name="ct6"; mkdir -p /etc/ipset_configs; ipset flush $ipset_name; url=$(cat /etc/ipset_configs/ipset_$ipset_name.url); curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; curl -s $url > /etc/ipset_configs/ipset_$ipset_name.txt
0 18 * * * ipset_name="cmcc4"; mkdir -p /etc/ipset_configs; ipset flush $ipset_name; url=$(cat /etc/ipset_configs/ipset_$ipset_name.url); curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; curl -s $url > /etc/ipset_configs/ipset_$ipset_name.txt
0 19 * * * ipset_name="cnc4"; mkdir -p /etc/ipset_configs; ipset flush $ipset_name; url=$(cat /etc/ipset_configs/ipset_$ipset_name.url); curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; curl -s $url > /etc/ipset_configs/ipset_$ipset_name.txt
0 20 * * * ipset_name="ct4"; mkdir -p /etc/ipset_configs; ipset flush $ipset_name; url=$(cat /etc/ipset_configs/ipset_$ipset_name.url); curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; curl -s $url > /etc/ipset_configs/ipset_$ipset_name.txt
```

```
0 21 * * * ipset_name="cn6"; mkdir -p /etc/ipset_configs; ipset flush $ipset_name; url=$(cat /etc/ipset_configs/ipset_$ipset_name.url); curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; curl -s $url > /etc/ipset_configs/ipset_$ipset_name.txt
0 22 * * * ipset_name="cn4"; mkdir -p /etc/ipset_configs; ipset flush $ipset_name; url=$(cat /etc/ipset_configs/ipset_$ipset_name.url); curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; curl -s $url > /etc/ipset_configs/ipset_$ipset_name.txt
```
# 原理解析

## 命令 1

初始化变量：定义 ipset_name 为 "NAME6"，url 为 "URL"。

创建配置目录：确保 /etc/ipset_configs 目录存在。

保存URL：将 url 写入配置文件 /etc/ipset_configs/ipset_NAME6.url。

检查并销毁旧的ipset：如果 ipset_name 已存在，则销毁它。

创建新的ipset：创建一个新的 IPv6 地址集合。

添加地址：从 url 获取地址列表并逐个添加到 ipset_name。

创建init.d脚本：如果 /etc/init.d/ipset_NAME6 不存在，创建它。

脚本内容包括：启动服务（从保存的配置恢复或重新创建ipset并从URL重新添加地址）、停止服务（保存当前ipset状态并销毁）、服务触发器。

设置脚本权限并启用：赋予脚本可执行权限并启用它。

保存地址列表：从 url 获取地址列表并保存到 /etc/ipset_configs/ipset_NAME6.txt。

## 命令 2

初始化变量：定义 ipset_name 为 "NAME"。

创建配置目录：确保 /etc/ipset_configs 目录存在。

清空ipset：清空名为 ipset_name 的地址集合。

读取URL：从配置文件 /etc/ipset_configs/ipset_NAME.url 读取URL。

添加地址：从 url 获取地址列表并逐个添加到 ipset_name。

保存地址列表：从 url 获取地址列表并保存到 /etc/ipset_configs/ipset_NAME.txt。

## 总结

命令1提供了一个完整的服务管理方案，包括自动创建、更新和保存ipset，以及服务的启停管理。适用于需要持久化ipset配置和服务管理的场景。

命令2更简洁，适用于仅需临时更新ipset地址集合的场景，不涉及服务的持久化管理。

两者的共同点是都从URL获取地址并更新到指定的ipset，但1在功能上更为全面，适用于复杂场景，而2更为简洁，适用于简单场景。2必须依赖于1才能使用。
