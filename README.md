# Update-the-IPset

无需安装mwan3 helper，通过终端添加ipset并自动更新，设置持久化和开机自启。在同一个IPset内，不能同时调用IPv4和IPv6，因此选择分开执行。

使用前请确保安装了依赖

```
opkg update && opkg install ipset busybox coreutils
```

# 终端内首次运行

```
mkdir -p /etc/ipset_configs && echo -e '#!/bin/sh\n\nCFG_DIR="/etc/ipset_configs"\n\nvalidate_input() {\n    if [[ ! "$name" =~ ^[a-zA-Z0-9_-]+$ ]]; then\n        echo "Invalid name"; exit 1\n    fi\n    if [[ "$url" != "" && ! "$url" =~ ^https?:// ]]; then\n        echo "Invalid url"; exit 1\n    fi\n    if [ "$type" != "4" ] && [ "$type" != "6" ]; then\n        echo "Invalid type"; exit 1\n    fi\n}\n\ndownload_file() {\n    local retries=3\n    local count=0\n    while [ $count -lt $retries ]; do\n        wget -qO $1 $2\n        if [ -s $1 ]; then\n            return 0\n        fi\n        count=$((count + 1))\n        sleep 1\n    done\n    return 1\n}\n\nadd_ipset() {\n    validate_input\n    family="inet$( [ "$type" -eq 6 ] && echo "6")"\n    f=$CFG_DIR/${name}.txt\n    rm -f $f\n    if ! download_file $f $url; then echo "Failed to download or empty file"; exit 1; fi\n    ipset create $name hash:net family $family -exist\n    ipset flush $name\n    sed -e "s/^/add $name /" $f | ipset restore -!\n    grep -v "^$name " $CFG_DIR/ipset_list > /tmp/ipset_list\n    mv /tmp/ipset_list $CFG_DIR/ipset_list\n    echo "$name $url $type" >> $CFG_DIR/ipset_list\n}\n\nclear_and_update_ipset() {\n    validate_input\n    f=$CFG_DIR/${name}.txt\n    > $f\n    url=$(grep "^$name " $CFG_DIR/ipset_list | awk "{print \$2}")\n    if ! download_file $f $url; then echo "Failed to download or empty file"; exit 1; fi\n    ipset flush $name\n    sed -e "s/^/add $name /" $f | ipset restore -!\n}\n' > /etc/ipset_configs/vars.sh && > /etc/ipset_configs/ipset_list && echo -e '#!/bin/sh /etc/rc.common\n\nSTART=99\nstart() {\n    . /etc/ipset_configs/vars.sh\n    while IFS=" " read -r name url type; do\n        family="inet$( [ "$type" -eq 6 ] && echo "6")"\n        f=$CFG_DIR/${name}.txt\n        [ -f $f ] && ipset create $name hash:net family $family -exist && ipset flush $name && sed -e "s/^/add $name /" $f | ipset restore -!\n    done < $CFG_DIR/ipset_list\n}' > /etc/init.d/ipset_load && chmod +x /etc/init.d/ipset_load && /etc/init.d/ipset_load enable
```

创建 /etc/ipset_configs 目录。用于缓存IP段，会将IPset的IP段保存至对应的txt文件中。

创建 vars.sh 文件。之后写入相关变量和函数，用于后续命令来调用它们。

创建 ipset_list 文件。用于保存IPset名称，及其对应的URL

创建 /etc/init.d/ipset_load 文件。用于开机启动，并将缓存内的IP段导入到IPset之中。

# 添加IPset并导入

```
name="NAME"; url="URL"; type="IP"; . /etc/ipset_configs/vars.sh; add_ipset
```

> 将`NAME`、`URL`、`IP`自定义。其中 `NAME` 对应IPset名称,只能包含字母（不区分大小写）、数字、下划线 (_) 和短横线 (-)。 `URL` 是其对应链接，必须以 `http://` 或 `https://` 开头。 `IP` 只能填写 `4` 或 `6` ，对应IPv4或IPv6。IP段被缓存在/etc/ipset_configs的txt文件之中

# 定时更新

后续只需要运行以下命令就可以更新IPset

```
name="NAME"; . /etc/ipset_configs/vars.sh; clear_and_update_ipset
```

> 只需要修改`NAME`即可

在命令前面加入（* * * * * ），就可以在计划任务中定期运行，注意空格位置

比如

```
0 20 * * * name="cn6"; . /etc/ipset_configs/vars.sh; clear_and_update_ipset
```

意味着每天`20`点自动更新`cn6`的IP段

# 命令

## cn6

```
name="cn6"; url="https://mirror.ghproxy.com/https://raw.githubusercontent.com/mayaxcn/china-ip-list/master/chnroute_v6.txt"; type="6"; . /etc/ipset_configs/vars.sh; add_ipset
```

## cmcc6

```
name="cmcc6"; url="https://gaoyifan.github.io/china-operator-ip/cmcc6.txt"; type="6"; . /etc/ipset_configs/vars.sh; add_ipset
```

## cnc6

```
name="cnc6"; url="https://gaoyifan.github.io/china-operator-ip/unicom6.txt"; type="6"; . /etc/ipset_configs/vars.sh; add_ipset
```

## ct6

```
name="ct6"; url="https://gaoyifan.github.io/china-operator-ip/chinanet6.txt"; type="6"; . /etc/ipset_configs/vars.sh; add_ipset
```

## cn4

```
name="cn4"; url="https://mirror.ghproxy.com/https://raw.githubusercontent.com/mayaxcn/china-ip-list/master/chnroute.txt"; type="4"; . /etc/ipset_configs/vars.sh; add_ipset
```

## cmcc4

```
name="cmcc4"; url="https://gaoyifan.github.io/china-operator-ip/cmcc.txt"; type="4"; . /etc/ipset_configs/vars.sh; add_ipset
```

## cnc4

```
name="cnc4"; url="https://gaoyifan.github.io/china-operator-ip/unicom.txt"; type="4"; . /etc/ipset_configs/vars.sh; add_ipset
```

## ct4

```
name="ct4"; url="https://gaoyifan.github.io/china-operator-ip/chinanet.txt"; type="4"; . /etc/ipset_configs/vars.sh; add_ipset
```

# 计划任务

```
0 15 * * * name="cmcc6"; . /etc/ipset_configs/vars.sh; clear_and_update_ipset
0 16 * * * name="cnc6"; . /etc/ipset_configs/vars.sh; clear_and_update_ipset
0 17 * * * name="ct6"; . /etc/ipset_configs/vars.sh; clear_and_update_ipset
0 18 * * * name="cmcc4"; . /etc/ipset_configs/vars.sh; clear_and_update_ipset
0 19 * * * name="cnc4"; . /etc/ipset_configs/vars.sh; clear_and_update_ipset
0 20 * * * name="ct4"; . /etc/ipset_configs/vars.sh; clear_and_update_ipset

```

```
0 21 * * * name="cn6"; . /etc/ipset_configs/vars.sh; clear_and_update_ipset
0 22 * * * name="cn4"; . /etc/ipset_configs/vars.sh; clear_and_update_ipset

```
