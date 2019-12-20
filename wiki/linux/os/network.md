---
title: linux 网络
date: 2019-11-28
tags:
categories: linux
---

### iproute2

### dhcpcd

DNS搜索域: 如果DNS解析不了，会加上域再解析一遍, 一般可以不填

### wpa_supplicant 和 wpa_cli

wpa_supplicant是一个连接、配置WIFI的工具，它主要包含wpa_supplicant与wpa_cli两个程序. 可以通过wpa_cli来进行WIFI的配置与连接,前提要保证wpa_supplicant正常启动

启动wpa_supplicant应用
~~~
wpa_supplicant -D nl80211 -i wlan0 -c /etc/wpa_supplicant.conf -B
~~~
* -D 驱动程序名称（可以是多个驱动程序：nl80211，wext）
* -i 接口名称
* -c 配置文件　
* -B 在后台运行守护进程

启动wpa_cli

~~~
wpa_cli -i wlan0 scan         　//搜索附件wifi热点
wpa_cli -i wlan0 scan_result 　 //显示搜索wifi热点
wpa_cli -i wlan0 status        //当前WPA/EAPOL/EAP通讯状态
wpa_cli -i wlan0 ping          //pings wpa_supplicant
~~~

添加新的连接

~~~
wpa_cli -i wlan0 add_network   //添加一个网络连接,会返回<network id> 
wpa_cli set_network <network id>  ssid '"name"'  //ssid名称 
wpa_cli set_network <network id>  psk '“psk”'　　//密码
wpa_cli set_network <network id>  scan_ssid 1   
wpa_cli set_network <network id>  priority  1   //优先级
~~~

保存连接
~~~
wpa_cli -i wlan0 save_config   //信息保存到默认的配置文件中
~~~

断开连接
~~~
wpa_cli -i wlan0 disable_network <network id> 
~~~

连接已有连接

~~~
wpa_cli -i wlan0 list_network  //列举保存过得连接
wpa_cli -i wlan0 select_network  <network id>  //连接指定的ssid 
wpa_cli -i wlan0 enable_network  <network id>  //使能制定的ssid 
~~~

配置文件示例:

~~~
ctrl_interface=/var/run/wpa_supplicant/
ap_scan=1
network={
    scan_ssid=1
    ssid="xxxx"
    psk="xxxx"
    bssid=
    priority=2
}
~~~