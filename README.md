# Update-the-IPset-of-MWAN3-helper

mwan3 helper的IPset中的IP地址段不会进行更新。通过加入到计划任务中，可以实现定时更新IP段。

# 计划任务更新ipset

```
0 16 * * * curl -s https://gaoyifan.github.io/china-operator-ip/cmcc.txt | tee /dev/stderr | cat - <(curl -s https://gaoyifan.github.io/china-operator-ip/cmcc6.txt) > /etc/mwan3helper/cmcc.txt
0 17 * * * curl -s https://gaoyifan.github.io/china-operator-ip/unicom.txt | tee /dev/stderr | cat - <(curl -s https://gaoyifan.github.io/china-operator-ip/unicom6.txt) > /etc/mwan3helper/unicom_cnc.txt
0 18 * * * curl -s https://gaoyifan.github.io/china-operator-ip/chinanet.txt | tee /dev/stderr | cat - <(curl -s https://gaoyifan.github.io/china-operator-ip/chinanet6.txt) > /etc/mwan3helper/chinatelecom.txt
0 19 * * * curl -s https://cdn.jsdelivr.net/gh/Loyalsoldier/v2ray-rules-dat@release/gfw.txt > /etc/mwan3helper/gfw.txt
0 20 * * * curl -s https://raw.githubusercontent.com/mayaxcn/china-ip-list/master/chnroute.txt | tee /dev/stderr | cat - <(curl -s https://raw.githubusercontent.com/mayaxcn/china-ip-list/master/chnroute_v6.txt) > /etc/mwan3helper/all_cn.txt
```

分别在每天的固定时间更新三大运营商、GFWlist、中国大陆IP段，合并了IPv4和IPv6。

比如每天`16`点从`https://gaoyifan.github.io/china-operator-ip/cmcc.txt`和`https://gaoyifan.github.io/china-operator-ip/cmcc6.txt`合并后替换`/etc/mwan3helper/cmcc.txt`的内容
