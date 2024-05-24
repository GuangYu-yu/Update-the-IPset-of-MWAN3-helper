# Update-the-IPset-of-MWAN3-helper

定时更新mwan3 helper的IP段,并添加三大运营商的IPv6-IPset。定时更新GFWlist和中国大陆IPv4和IPv6地址。

# 计划任务更新ipset

```
0 15 * * * curl -s https://gaoyifan.github.io/china-operator-ip/cmcc.txt | tee /dev/stderr > /etc/mwan3helper/cmcc.txt

0 16 * * * curl -s https://gaoyifan.github.io/china-operator-ip/unicom.txt > /etc/mwan3helper/unicom_cnc.txt

0 17 * * * curl -s https://gaoyifan.github.io/china-operator-ip/chinanet.txt > /etc/mwan3helper/chinatelecom.txt

0 18 * * * curl -s https://cdn.jsdelivr.net/gh/Loyalsoldier/v2ray-rules-dat@release/gfw.txt > /etc/mwan3helper/gfw.txt

0 19 * * * curl -s https://raw.githubusercontent.com/mayaxcn/china-ip-list/master/chnroute.txt > /etc/mwan3helper/all_cn.txt

0 20 * * * ipset -L cn6 >/dev/null 2>&1 && ipset flush cn6 || ipset create cn6 hash:net family inet6; curl -s https://raw.githubusercontent.com/mayaxcn/china-ip-list/master/chnroute_v6.txt | xargs -n1 ipset add cn6

0 21 * * * ipset -L cmcc6 >/dev/null 2>&1 && ipset flush cmcc6 || ipset create cmcc6 hash:net family inet6; curl -s https://gaoyifan.github.io/china-operator-ip/cmcc6.txt | xargs -n1 ipset add cmcc6

0 22 * * * ipset -L cnc6 >/dev/null 2>&1 && ipset flush cnc6 || ipset create cnc6 hash:net family inet6; curl -s https://gaoyifan.github.io/china-operator-ip/unicom6.txt | xargs -n1 ipset add cnc6

0 23 * * * ipset -L ct6 >/dev/null 2>&1 && ipset flush ct6 || ipset create ct6 hash:net family inet6; curl -s https://gaoyifan.github.io/china-operator-ip/chinanet6.txt | xargs -n1 ipset add ct6
```

每天`15`点从`https://gaoyifan.github.io/china-operator-ip/cmcc.txt`覆盖`/etc/mwan3helper/cmcc.txt`的内容

每天`20`点检测`cn6`这个IPset是否存在，存在且非空则清空Ipset的内容，不存在则创建`cn6`。之后从`https://raw.githubusercontent.com/mayaxcn/china-ip-list/master/chnroute_v6.txt`获取IPv6地址，并加入到`cn6`之中。
