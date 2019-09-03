---
title: 脚本自动化关闭Zabbix中Down的主机
date: 2019-09-03 18:47
categories:
  - linux
  - zabbix
---
以下我使用Python通过Py-Zabbix的API来进行关闭Zabbix中已经Down的主机。基本思路是
1.  获得前所属组的IP
2.  通过telnet去检验主机是否Down
3.  关闭主机

如下是脚本初始小damo，*萌新一定要先测试脚本是不是符合你的生产环境后再在正式环境下执行*
```python
#！/bin/env python


import telnetlib
import pyzabbix
from pyzabbix import ZabbixAPI
import os

web = input("Input Your Zabbix-Web address:")
username = input("API User :")
passwd = input("Password:")
groupids = input("HostGroup1:15\nHostGroup2:16\nInput Your Group: ")
#check Your Password
try:
    zapi = ZabbixAPI(url=web, user=username, password=passwd)
    print("认证成功")
    zapi.do_request('user.logout')
except:
    print("用户认证错误！！脚本退出")
    os._exit(0)
#Get Host IP From Zabbix
def ZabbixIP():
    zapi = ZabbixAPI(url=web, user=username, password=passwd)
    #获得输出结果的个数
    result1 = zapi.do_request("host.get", {"groupids":groupids, "output": ["name"], 'countOutput': True})
    #输出结果带IP
    result2 = zapi.do_request("host.get", {"groupids":groupids, "output": ["name"], "selectInterfaces": ["interfaces", "ip"]})
    host_count = int(result1.get(u'result'))
    #初始化IPList，一会拿来装IP
    IPList = []
    for i in range(host_count):
        num = i
        #分别输出元组
        VeeeHostList = result2.get(u'result')[num]
        # print(VeeeHostList)
        #得到一个包含ip的元组 eg：{interfaces:["ip":"0.0.0.0"]}
        HostIP = VeeeHostList.get('interfaces')
        #到元组里面的数列["ip":"0.0.0.0"]
        IpTup = HostIP[0]
        Ip = IpTup.get(r"ip")
        #print(Ip)
        IPList.append(Ip)
    zapi.do_request('user.logout')
    return IPList
#做telnet 测试端口是否开放。即检测服务器是否在线，不在线则关闭主机
def telenetlib():
    WrongIpList = []
    RightIpList = []
    for ip in IpGroup:
        try:
            tn = telnetlib.Telnet(host=ip,port=22,timeout=5)
            print("狗屎笨笨")
            RightIpList.append(ip)
        except:
            print("there is sometime wrong with " + ip + "!!!")
            WrongIpList.append(ip)
    return WrongIpList,RightIpList
def UpdateHost():
    zapi = ZabbixAPI(url=web, user=username, password=passwd)
    for Winterface in IpWrongGroup:
        result3 = zapi.do_request("host.get",{"output":"hostid","selectInterfaces":["interfaces","ip"],"filter":{"ip":Winterface}})
        result4 = str(result3.get(u'result')).split('\'')
        hostid = result4[3]
        zapi.do_request("host.update",{"hostid":hostid,"status":1})
    for Rinterface in IpRightGroup:
        result3 = zapi.do_request("host.get",{"output":"hostid","selectInterfaces":["interfaces","ip"],"filter":{"ip":Rinterface}})
        result4 = str(result3.get(u'result')).split('\'')
        hostid = result4[3]
        zapi.do_request("host.update",{"hostid":hostid,"status":0})
    zapi.do_request('user.logout')
IpGroup = ZabbixIP()
IpWrongGroup,IpRightGroup = telenetlib()
UpdateHost()

```
敬上，溜了。
有啥问题，你可以联系博主 Telegram：@tingbob