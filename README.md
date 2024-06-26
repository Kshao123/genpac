# GenPac
**基于[gfwlist][gfw-list-uri]的多种代理软件配置文件生成工具，支持自定义规则，目前可生成的格式有pac, dnsmasq, wingy。**
> 基于仓库 [JinnLynn/genpac][genpac]（具体使用可去原 [README](./ORIGINAL_README.md) 查找），参考 [petronny/gfwlist2pac][gfwlist2pac] 编写相关代码。

## Diff
**genpac/pysocks/socks.py**
```diff
- from collections import Callable
+ from collections.abc import Callable
```

### 安装
**方案一**
```shell
# 安装或更新
$ pip install -U genpac
# 或从github安装更新开发版本
$ pip install -U https://github.com/JinnLynn/genpac/archive/dev.zip

# 卸载
$ pip uninstall genpac
```
**方案二**
```shell
$ git clone https://github.com/Kshao123/gfwlist2pac.git
$ cd gfwlist2pac
$ python setup.py install

# 快速测试，原始文件和结果可在 demo 目录下查看（pac-proxy 采用 clash 默认代理配置，可根据不同情况自行修改）
# "--gfwlist-url -" 表示不使用远程 gfwlist，"--gfwlist-local demo/gfwlist.txt" 表示使用本地 gfwlist
$ genpac --pac-proxy "SOCKS5 127.0.0.1:7890; SOCKS 127.0.0.1:7890; DIRECT;" --gfwlist-url - --gfwlist-local demo/gfwlist.txt -o demo/gfwlist.pac

# 或使用自己的规则来覆盖 "--user-rule-from demo/user-rules.txt"
# 根据 http://adblockplus.org/en/filters 规则来定义，可根据 demo/user-rules.txt 查看
# 简单理解既：不想被 proxy：@@||openai.com，想被 proxy：||baidu.com
$ genpac --pac-proxy "SOCKS5 127.0.0.1:7890; SOCKS 127.0.0.1:7890; DIRECT;" --gfwlist-url - --gfwlist-local demo/gfwlist.txt -o demo/gfwlist.pac --user-rule-from demo/user-rules.txt
```


### 使用方法

```
genpac [-v] [-h] [--init [PATH]] [--format {pac,dnsmasq,wingy}]
       [--gfwlist-url URL] [--gfwlist-proxy PROXY]
       [--gfwlist-local FILE] [--gfwlist-update-local]
       [--gfwlist-disabled] [--gfwlist-decoded-save FILE]
       [--user-rule RULE] [--user-rule-from FILE]
       [--template FILE] [-o FILE] [-c FILE]
       [--pac-proxy PROXY] [--pac-precise] [--pac-compress]
       [--dnsmasq-dns DNS] [--dnsmasq-ipset IPSET]
       [--wingy-adapter-opts OPTS] [--wingy-rule-adapter-id ID]

获取gfwlist生成多种格式的翻墙工具配置文件, 支持自定义规则

optional arguments:
  -v, --version         版本信息
  -h, --help            帮助信息
  --init [PATH]         初始化配置和用户规则文件

通用参数:
  --format {pac,dnsmasq,wingy}
                        生成格式, 只有指定了格式, 相应格式的参数才作用
  --gfwlist-url URL     gfwlist网址，无此参数或URL为空则使用默认地址, URL为-则不在线获取
  --gfwlist-proxy PROXY
                        获取gfwlist时的代理, 如果可正常访问gfwlist地址, 则无必要使用该选项
                        格式为 "代理类型 [用户名:密码]@地址:端口" 其中用户名和密码可选, 如:
                          SOCKS5 127.0.0.1:8080
                          SOCKS5 username:password@127.0.0.1:8080
  --gfwlist-local FILE  本地gfwlist文件地址, 当在线地址获取失败时使用
  --gfwlist-update-local
                        当在线gfwlist成功获取且--gfwlist-local参数存在时, 更新gfwlist-local内容
  --gfwlist-disabled    禁用gfwlist
  --gfwlist-decoded-save FILE
                        保存解码后的gfwlist, 仅用于测试
  --user-rule RULE      自定义规则, 允许重复使用或在单个参数中使用`,`分割多个规则，如:
                          --user-rule="@@sina.com" --user-rule="||youtube.com"
                          --user-rule="@@sina.com,||youtube.com"
  --user-rule-from FILE
                        从文件中读取自定义规则, 使用方法如--user-rule
  -o FILE, --output FILE
                        输出到文件, 无此参数或FILE为-, 则输出到stdout
  -c FILE, --config-from FILE
                        从文件中读取配置信息
  --template FILE       自定义模板文件

PAC:
  通过代理自动配置文件（PAC）系统或浏览器可自动选择合适的代理服务器

  --pac-proxy PROXY     代理地址, 如 SOCKS5 127.0.0.1:8080; SOCKS 127.0.0.1:8080
  --pac-precise         精确匹配模式
  --pac-compress        压缩输出
  -p PROXY, --proxy PROXY
                        已弃用参数, 等同于--pac-proxy, 后续版本将删除, 避免使用
  -P, --precise         已弃用参数, 等同于--pac-precise, 后续版本将删除, 避免使用
  -z, --compress        已弃用参数, 等同于--pac-compress, 后续版本将删除, 避免使用

DNSMASQ:
  Dnsmasq配合iptables ipset可实现基于域名的自动直连或代理

  --dnsmasq-dns DNS     生成规则域名查询使用的DNS服务器，格式: HOST#PORT
                        默认: 127.0.0.1#53
  --dnsmasq-ipset IPSET
                        转发使用的ipset名称, 默认: GFWLIST

WINGY:
  Wingy是iOS下基于NEKit的代理App

  --wingy-adapter-opts OPTS
                        adapter选项, 选项间使用`,`分割, 多个adapter使用`;`分割, 如:
                          id:ap1,type:http,host:127.0.0.1,port:8080;id:ap2,type:socks5,host:127.0.0.1,port:3128
  --wingy-rule-adapter-id ID
                        生成规则使用的adapter ID
```
### 示例

```
# 从gfwlist生成代理信息为SOCKS5 127.0.0.1:1080的PAC文件
genpac --format=pac --pac-proxy="SOCKS5 127.0.0.1:1080"

# 从~/config.ini读取配置生成
genpac --config-from=~/config.ini

# PAC格式 压缩
genpac --format=pac --pac-proxy="SOCKS5 127.0.0.1:1080" --pac-compress

# PAC格式 精确匹配模式
genpac --format=pac --pac-proxy="SOCKS5 127.0.0.1:1080" --pac-precise

# PAC格式 自定义规则
genpac --format=pac --pac-proxy="SOCKS5 127.0.0.1:1080" "SOCKS5 127.0.0.1:1080" --user-rule="||example.com" --user-rule-from=~/user-rule.txt
genpac --config-from=~/config.ini --pac-proxy="SOCKS5 127.0.0.1:1080" --user-rule="||example.com" --user-rule-from=~/user-rule.txt

# PAC格式 多个自定义规则文件
genpac --format=pac --pac-proxy="SOCKS5 127.0.0.1:1080" --user-rule="||example.com" --user-rule="||example2.com" --user-rule-from=~/user-rule.txt,~/user-rule2.txt

# PAC格式 使用HTTP代理127.0.0.1:8080获取在线gfwlist文件
genpac --format=pac --pac-proxy="SOCKS5 127.0.0.1:1080" --gfwlist-proxy="PROXY 127.0.0.1:8080"

# PAC格式 如果在线gfwlist获取失败使用本地文件，如果在线gfwlist获取成功更新本地gfwlist文件
genpac --format=pac --pac-proxy="SOCKS5 127.0.0.1:1080" --gfwlist-local=~/gfwlist.txt --update-gfwlist-local

# PAC格式 忽略gfwlist，仅使用自定义规则
genpac --format=pac --pac-proxy="SOCKS5 127.0.0.1:1080" --gfwlist-disabled --user-rule-from=~/user-rule.txt
genpac --format=pac --pac-proxy="SOCKS5 127.0.0.1:1080" --gfwlist-url=- --user-rule-from=~/user-rule.txt

# DNSMASQ WINGY格式同样可以使用上述PAC格式中关于gfwlist和自定义规则的参数

# DNSMASQ格式
genpac --format=dnsmasq --dnsmasq-dns="127.0.0.1#53" --dnsmasq-ipset="ipset-name"

# WINGY格式 使用默认模板生成
genpac --format=wingy --wingy-opts="id:do-ss,type:ss,host:192.168.100.1,port:8888,method:bf-cfb,password:test" --wingy-rule-adapter-id=do-ss

# WINGY格式 使用自定义模板
genpac --format=wingy --template=/sample/wingy-tpl.yaml
```


[gfw-list-uri]: https://github.com/gfwlist/gfwlist
[genpac]: https://github.com/JinnLynn/genpac
[gfwlist2pac]: https://github.com/petronny/gfwlist2pac
