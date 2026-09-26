---
layout: post
title: 9.15下午——WLAN（AC+AP）
abbrlink: c21e9849e1ea4e6fbf4bde29aeeb8bdf
tags: []
categories:
  - 华为认证
  - 国科HCIE
date: 1789453900711
updated: 1789644651576
---

## 一、AP

**1、什么是AP：**\
    Access Point是指为终端提供无线接入服务的无线接入点，简称AP\
    Wi-Fi是AP使用的主流通信技术，而WLAN是由AP、AC（Access Controller）等设备组成的无线局域网络\
**2、FAT AP：**\
    又称为胖AP，独立完成Wi-Fi覆盖，不需要另外部署管控设备\
    由于FAT AP独自控制用户的接入，用户无法在FAT AP之间实现无线漫游，只有在FAT AP覆盖范围内才能使用Wi-Fi网络\
    FAT AP可以手动配置\
    优点：造价便宜、部署方便（空间不大）\
    缺点：配置数量多、排错故障电难发现\
**3、漫游：**\
    当WiFi的名字和密码一致时，随着用户的位置变化，能够自动切换到不同AP\
**4、AC+FIT AP：**\
    AC是指（WLAN Access Controller，简称WAC或AC）无线接入控制器，是无线局域网中的核心角色，一般部署于汇聚层，可以实现对无线接入点（Access Point，简称AP）的批量业务配置和管理 \
    同时，因为用户的接入认证可以由AC统一管理，所以用户可以在AP间实现无线漫游\
    FIT AP，也称瘦AP，不可以手动配置，必须AC控制下发\
    优点：管理容易，配置模板统一发布\
    缺点：造价贵，需要额外采购AP，已经对应设备数量的license授权10.1.200.2910.1.200.29

## 二、转发模式

<img src="/resources/e4530ecbb4734d8da38d655868eed602.png" alt="d6e4a0d35594e925364eb3522793af0e.png" width="722" height="356" class="jop-noMdConv"> 

**1、隧道转发**\
    capwap vlanif100——存在一个隧道学习AC提供的配置\
    要先找到AC的位置才能发DHCP\
    隧道转发需要在AC和DHCP之间放业务vlan\
    看<span style="color: rgb(53, 152, 219);">蓝色</span>的标注在AC2和LSW3之间的\
**2、直接转发**\
    直接转发业务直接访问网关上网，不需要绕AC\
    直接转发需要在AP和DHCP之间放业务vlan\
    看<span style="color: rgb(45, 194, 107);">绿色</span>的标注在LSW3和AP7之间的

## 三、配置实例与步骤

<img src="/resources/123d498d3de140a5994d4da55c6a9109.png" alt="0c5ef437a68a356b96adf462a7aaf76b.png" width="463" height="518" class="jop-noMdConv">

**1、AP上线**\
（1）AP获取DHCP分配的IP\
       AC部署DHCP服务器保证能够让AP获取IP\
    （沿路交换机放行vlan，交换机与AC互联配置trunk，交换机与AP互联配置access）\
       AP获取IP后\
       AC配置capwap source interface vlanif 100 //分配DHCP的接口\
     （由于安全考虑，需要对AP进行身份验证）\
（2）AC进入WLAN设置\
        ap-id 0\~8191 ap-mac xxxx:xxxx:xxxx\
           其中AP的MAC可以去到上行接入交换机部署\
           lldp enable\
           display lldp neighbor 进行查询AP的MAC\
        最后在AC执行display ap all，如果发现状态是nor就是成功

**2、配置上网模板**\
  （1）SSID——wifi的名字\
           wlan视图下\
          创建ssid-profile xxx //xxx为用户希望的“模板”名字\
             ssid yyyy //yyyy为当前用户希望在连接wifi的时候看到的wifi名字\
  （2）创建security-profile，在其中设置wifi的密码\
           security-profile name xxx //xxx为用户希望的“模板”名字\
                security wpa-wpa2 psk pass-phrase Huawe\@123 aes //其中Huawe\@123是该wifi的密码\
               ① wpa-wpa2 //wifi联盟创造的一个加密方法\
               ② psk //dot1协和psk，服务器提供准入信息以及直接密文认证，由于当前没有服务器，所以用PSK\
  （3）vlan——提供上网的IP\
          需要给用户创建相关vlan，并且该vlan提供DHCP的服务\
  （4）转发模式——隧道转发/直接转发

**3、调用AP模板进行使用**\
   （1）vap-profile name xxx //xxx为用户希望的“模板”名字\
            ssid-profile xxx\
            security-profile xxx\
            forward-mode tunnel / direct-forward\
            service-vlan vlan-pool Guest / vlan-id xx\
   （2）ap-id x\
           vap-profile xxx wlan 1~~16 radio all / 0，1，2\
           wlan 1~~16（进程号）\
           radio all/0/1/2  0=2.4Ghz （范围广，穿透差）； 1=5Ghz 1版本（穿透强，覆盖略少）；2=5Ghz2版本

 
