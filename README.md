# 屏蔽方法

### Windows

下载`rpglist.json`，导入火绒或者其他类似能提供IP过滤名单的软件，不再赘述



### Linux

（提供Ubuntu/Mint的方案，其他distro请自行对号入座）


1. 下载屏蔽IP名单

   ```bash
   curl https://raw.githubusercontent.com/typowritter/block-l4d2-rpg-servers/master/rpglist.json -o rpglist.json
   ```

2. 安装命令行JSON parser和ipset

   ```bash
   sudo apt install jq ipset
   ```

3. 新建屏蔽列表 l4d2-rpg-blacklist

   ```bash
   sudo ipset create l4d2-rpg-blacklist hash:ip hashsize 4096
   ```

4. 解析名单里的所有IP并加入屏蔽列表

   ```bash
   jq -r '.data[].raddr' rpglist.json | xargs -L1 sudo ipset add l4d2-rpg-blacklist
   ```

5. 启用屏蔽列表

   ```bash
   sudo iptables -I OUTPUT -p UDP -m set --match-set l4d2-rpg-blacklist dst -j DROP
   ```

好了，现在这些RPG服就被全部屏蔽了。如果要加入新的IP，直接`sudo ipset add l4d2-rpg-blacklist xx.xx.xx.xx`



但是还有个问题，由于iptables和ipset规则默认并不是持久化的，会在重启后丢失。可以有两种方法解决：

- 方法一（**推荐**，方便更新列表）：将上述步骤保存为Shell Script，设置每次开机时执行
- 方法二（如果你知道自己在做什么）：建立一个持久化服务，在每次退出时保存iptables和ipset规则，并在启动时恢复。参考：[Using ipset to block IP addresses - firewall](https://confluence.jaytaala.com/display/TKB/Using+ipset+to+block+IP+addresses+-+firewall)，以及[Make ip-tables (firewall) rules persistent](https://confluence.jaytaala.com/display/TKB/Make+ip-tables+(firewall)+rules+persistent)





