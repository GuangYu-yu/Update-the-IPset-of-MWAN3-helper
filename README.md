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
