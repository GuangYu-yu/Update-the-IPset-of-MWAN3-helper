# Update-the-IPset-of-MWAN3-helper

这个项目原本是为了定期更新mwan3 helper中的IP段的，但是效果并不好，因此采用别的方式使用IPset分流。无需安装mwan3 helper，通过终端添加ipset并自动更新，设置持久化和开机自启。在同一个IPset内，不能同时调用IPv4和IPv6，因此选择分开执行。mwan3 helper内的IP段从未更新且仅有IPv4地址，对于多宽带分流来说非常鸡肋。

使用时请确保存在`ipset`、`curl`、`xargs`依赖

# 终端内首次运行

## IPv6 IPset

```
ipset_name="NAME6"; url="URL"; echo $url > /etc/ipset_$ipset_name.url; { ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet6; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; } && [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nstart() {\n    ipset -L '"$ipset_name"' &>/dev/null && { ipset destroy '"$ipset_name"'; }\n    ipset create '"$ipset_name"' hash:net family inet6\n    curl -s $(cat /etc/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}\n}\nstop() {\n    ipset destroy '"$ipset_name"'\n}\nrestart() {\n    stop\n    start\n}\nboot() {\n    start\n}\ncase "$1" in\n    start|stop|restart|boot) $1 ;;\n    *) echo "Usage: $0 {start|stop|restart|boot}"; exit 1 ;;\nesac' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; } && { /etc/init.d/ipset_$ipset_name status &>/dev/null || /etc/init.d/ipset_$ipset_name start; }
```

## IPv4 IPset

```
ipset_name="NAME4"; url="URL"; echo $url > /etc/ipset_$ipset_name.url; { ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; } && [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nstart() {\n    ipset -L '"$ipset_name"' &>/dev/null && { ipset destroy '"$ipset_name"'; }\n    ipset create '"$ipset_name"' hash:net family inet\n    curl -s $(cat /etc/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}\n}\nstop() {\n    ipset destroy '"$ipset_name"'\n}\nrestart() {\n    stop\n    start\n}\nboot() {\n    start\n}\ncase "$1" in\n    start|stop|restart|boot) $1 ;;\n    *) echo "Usage: $0 {start|stop|restart|boot}"; exit 1 ;;\nesac' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; } && { /etc/init.d/ipset_$ipset_name status &>/dev/null || /etc/init.d/ipset_$ipset_name start; }
```

> 将`NAME4`、`NAME6`、`URL`自定义，注意：`IPv6 IPset`和`IPv4 IPset`并不一致。

# 定时更新

后续只需要运行以下两行命令就可以更新IPset

```
ipset_name="NAME"; ipset flush $ipset_name; curl -s $(cat /etc/ipset_$ipset_name.url) | xargs -n1 -I{} ipset add $ipset_name {}
```

> 只需要修改`NAME`即可，IPv4和IPv6的后续命令是通用的

在命令前面加入（* * * * * ），就可以在计划任务中定期运行，注意空格位置

比如

```
0 20 * * * ipset_name="cn6"; ipset flush $ipset_name; curl -s $(cat /etc/ipset_$ipset_name.url) | xargs -n1 -I{} ipset add $ipset_name {}
```

意味着每天`20`点自动更新`cn6`的IP段

# 命令

## cn6

```
ipset_name="cn6"; url="https://raw.githubusercontent.com/mayaxcn/china-ip-list/master/chnroute_v6.txt"; echo $url > /etc/ipset_$ipset_name.url; { ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet6; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; } && [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nstart() {\n    ipset -L '"$ipset_name"' &>/dev/null && { ipset destroy '"$ipset_name"'; }\n    ipset create '"$ipset_name"' hash:net family inet6\n    curl -s $(cat /etc/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}\n}\nstop() {\n    ipset destroy '"$ipset_name"'\n}\nrestart() {\n    stop\n    start\n}\nboot() {\n    start\n}\ncase "$1" in\n    start|stop|restart|boot) $1 ;;\n    *) echo "Usage: $0 {start|stop|restart|boot}"; exit 1 ;;\nesac' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; } && { /etc/init.d/ipset_$ipset_name status &>/dev/null || /etc/init.d/ipset_$ipset_name start; }
```

## cmcc6

```
ipset_name="cmcc6"; url="https://gaoyifan.github.io/china-operator-ip/cmcc6.txt"; echo $url > /etc/ipset_$ipset_name.url; { ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet6; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; } && [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nstart() {\n    ipset -L '"$ipset_name"' &>/dev/null && { ipset destroy '"$ipset_name"'; }\n    ipset create '"$ipset_name"' hash:net family inet6\n    curl -s $(cat /etc/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}\n}\nstop() {\n    ipset destroy '"$ipset_name"'\n}\nrestart() {\n    stop\n    start\n}\nboot() {\n    start\n}\ncase "$1" in\n    start|stop|restart|boot) $1 ;;\n    *) echo "Usage: $0 {start|stop|restart|boot}"; exit 1 ;;\nesac' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; } && { /etc/init.d/ipset_$ipset_name status &>/dev/null || /etc/init.d/ipset_$ipset_name start; }
```

## cnc6

```
ipset_name="cnc6"; url="https://gaoyifan.github.io/china-operator-ip/unicom6.txt"; echo $url > /etc/ipset_$ipset_name.url; { ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet6; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; } && [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nstart() {\n    ipset -L '"$ipset_name"' &>/dev/null && { ipset destroy '"$ipset_name"'; }\n    ipset create '"$ipset_name"' hash:net family inet6\n    curl -s $(cat /etc/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}\n}\nstop() {\n    ipset destroy '"$ipset_name"'\n}\nrestart() {\n    stop\n    start\n}\nboot() {\n    start\n}\ncase "$1" in\n    start|stop|restart|boot) $1 ;;\n    *) echo "Usage: $0 {start|stop|restart|boot}"; exit 1 ;;\nesac' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; } && { /etc/init.d/ipset_$ipset_name status &>/dev/null || /etc/init.d/ipset_$ipset_name start; }
```

## ct6

```
ipset_name="ct6"; url="https://gaoyifan.github.io/china-operator-ip/chinanet6.txt"; echo $url > /etc/ipset_$ipset_name.url; { ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet6; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; } && [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nstart() {\n    ipset -L '"$ipset_name"' &>/dev/null && { ipset destroy '"$ipset_name"'; }\n    ipset create '"$ipset_name"' hash:net family inet6\n    curl -s $(cat /etc/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}\n}\nstop() {\n    ipset destroy '"$ipset_name"'\n}\nrestart() {\n    stop\n    start\n}\nboot() {\n    start\n}\ncase "$1" in\n    start|stop|restart|boot) $1 ;;\n    *) echo "Usage: $0 {start|stop|restart|boot}"; exit 1 ;;\nesac' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; } && { /etc/init.d/ipset_$ipset_name status &>/dev/null || /etc/init.d/ipset_$ipset_name start; }
```

## cn4

```
ipset_name="cn4"; url="https://raw.githubusercontent.com/mayaxcn/china-ip-list/master/chnroute.txt"; echo $url > /etc/ipset_$ipset_name.url; { ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; } && [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nstart() {\n    ipset -L '"$ipset_name"' &>/dev/null && { ipset destroy '"$ipset_name"'; }\n    ipset create '"$ipset_name"' hash:net family inet\n    curl -s $(cat /etc/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}\n}\nstop() {\n    ipset destroy '"$ipset_name"'\n}\nrestart() {\n    stop\n    start\n}\nboot() {\n    start\n}\ncase "$1" in\n    start|stop|restart|boot) $1 ;;\n    *) echo "Usage: $0 {start|stop|restart|boot}"; exit 1 ;;\nesac' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; } && { /etc/init.d/ipset_$ipset_name status &>/dev/null || /etc/init.d/ipset_$ipset_name start; }
```

## cmcc4

```
ipset_name="cn4"; url="https://gaoyifan.github.io/china-operator-ip/cmcc.txt"; echo $url > /etc/ipset_$ipset_name.url; { ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; } && [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nstart() {\n    ipset -L '"$ipset_name"' &>/dev/null && { ipset destroy '"$ipset_name"'; }\n    ipset create '"$ipset_name"' hash:net family inet\n    curl -s $(cat /etc/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}\n}\nstop() {\n    ipset destroy '"$ipset_name"'\n}\nrestart() {\n    stop\n    start\n}\nboot() {\n    start\n}\ncase "$1" in\n    start|stop|restart|boot) $1 ;;\n    *) echo "Usage: $0 {start|stop|restart|boot}"; exit 1 ;;\nesac' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; } && { /etc/init.d/ipset_$ipset_name status &>/dev/null || /etc/init.d/ipset_$ipset_name start; }
```

## cnc4

```
ipset_name="cn4"; url="https://gaoyifan.github.io/china-operator-ip/unicom.txt"; echo $url > /etc/ipset_$ipset_name.url; { ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; } && [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nstart() {\n    ipset -L '"$ipset_name"' &>/dev/null && { ipset destroy '"$ipset_name"'; }\n    ipset create '"$ipset_name"' hash:net family inet\n    curl -s $(cat /etc/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}\n}\nstop() {\n    ipset destroy '"$ipset_name"'\n}\nrestart() {\n    stop\n    start\n}\nboot() {\n    start\n}\ncase "$1" in\n    start|stop|restart|boot) $1 ;;\n    *) echo "Usage: $0 {start|stop|restart|boot}"; exit 1 ;;\nesac' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; } && { /etc/init.d/ipset_$ipset_name status &>/dev/null || /etc/init.d/ipset_$ipset_name start; }
```

## ct4

```
ipset_name="cn4"; url="https://gaoyifan.github.io/china-operator-ip/chinanet.txt"; echo $url > /etc/ipset_$ipset_name.url; { ipset -L $ipset_name &>/dev/null && ipset destroy $ipset_name; ipset create $ipset_name hash:net family inet; curl -s $url | xargs -n1 -I{} ipset add $ipset_name {}; } && [[ ! -f /etc/init.d/ipset_$ipset_name ]] && { echo -e '#!/bin/sh /etc/rc.common\nSTART=99\nstart() {\n    ipset -L '"$ipset_name"' &>/dev/null && { ipset destroy '"$ipset_name"'; }\n    ipset create '"$ipset_name"' hash:net family inet\n    curl -s $(cat /etc/ipset_'$ipset_name'.url) | xargs -n1 -I{} ipset add '"$ipset_name"' {}\n}\nstop() {\n    ipset destroy '"$ipset_name"'\n}\nrestart() {\n    stop\n    start\n}\nboot() {\n    start\n}\ncase "$1" in\n    start|stop|restart|boot) $1 ;;\n    *) echo "Usage: $0 {start|stop|restart|boot}"; exit 1 ;;\nesac' > /etc/init.d/ipset_$ipset_name; chmod +x /etc/init.d/ipset_$ipset_name; /etc/init.d/ipset_$ipset_name enable; } && { /etc/init.d/ipset_$ipset_name status &>/dev/null || /etc/init.d/ipset_$ipset_name start; }
```

# 计划任务

```
0 15 * * * ipset_name="cn6"; ipset flush $ipset_name; curl -s $(cat /etc/ipset_$ipset_name.url) | xargs -n1 -I{} ipset add $ipset_name {}
0 16 * * * ipset_name="cmcc6"; ipset flush $ipset_name; curl -s $(cat /etc/ipset_$ipset_name.url) | xargs -n1 -I{} ipset add $ipset_name {}
0 17 * * * ipset_name="cnc6"; ipset flush $ipset_name; curl -s $(cat /etc/ipset_$ipset_name.url) | xargs -n1 -I{} ipset add $ipset_name {}
0 18 * * * ipset_name="ct6"; ipset flush $ipset_name; curl -s $(cat /etc/ipset_$ipset_name.url) | xargs -n1 -I{} ipset add $ipset_name {}
0 19 * * * ipset_name="cn4"; ipset flush $ipset_name; curl -s $(cat /etc/ipset_$ipset_name.url) | xargs -n1 -I{} ipset add $ipset_name {}
0 20 * * * ipset_name="cmcc4"; ipset flush $ipset_name; curl -s $(cat /etc/ipset_$ipset_name.url) | xargs -n1 -I{} ipset add $ipset_name {}
0 21 * * * ipset_name="cnc4"; ipset flush $ipset_name; curl -s $(cat /etc/ipset_$ipset_name.url) | xargs -n1 -I{} ipset add $ipset_name {}
0 22 * * * ipset_name="ct4"; ipset flush $ipset_name; curl -s $(cat /etc/ipset_$ipset_name.url) | xargs -n1 -I{} ipset add $ipset_name {}
```
