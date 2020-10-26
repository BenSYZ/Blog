# 树莓派开热点
本文主要参考 [树莓派官网](https://www.raspberrypi.org/documentation/configuration/wireless/access-point-routed.md)

[toc]

首先允许数据包转发
```config
# /etc/sysctl.d/routed-ap.conf
net.ipv4.ip_forward=1
```

## Applications
###  `hostapd`：提供 wifi 热点名等

```sh
sudo apt install hostapd
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
```


* 2.4GHz 配置
```config
country_code=CN
interface=wlan0
ssid=wifiname
hw_mode=g
channel=7
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=password
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

* [5GHz 配置](https://github.com/raspberrypi/linux/issues/2619) 

由于之前提到2.4GHz 和USB3.0 信号互相干扰。因此可以选自5GHz，注意着了`country_code`得等于`US`,不能`CN`

```config
# /etc/hostapd/hostapd.conf
ssid=wifiname
wpa_passphrase=password

country_code=US

interface=wlan0
driver=nl80211

wpa=2
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP

macaddr_acl=0

logger_syslog=0
logger_syslog_level=4
logger_stdout=-1
logger_stdout_level=0

hw_mode=a
wmm_enabled=1

# N
ieee80211n=1
require_ht=1
ht_capab=[MAX-AMSDU-3839][HT40+][SHORT-GI-20][SHORT-GI-40][DSSS_CCK-40]

# AC
ieee80211ac=1
require_vht=1
ieee80211d=0
ieee80211h=0
vht_capab=[MAX-AMSDU-3839][SHORT-GI-80]
vht_oper_chwidth=1
channel=36
vht_oper_centr_freq_seg0_idx=42
```



###  `dnsmasq`：给连接到的设备分配IP，以及提供DNS服务
绑定Mac部分[参考](https://cloud.tencent.com/developer/article/1174717)，此文讲了挺多关于dnsmasq 的配置的。

```sh
sudo apt install dnsmasq
```

```config
#/etc/dnsmasq.conf
interface=wlan0 # Listening interface
dhcp-range=10.0.1.2,10.0.1.20,255.255.255.0,24h
                # Pool of IP addresses served via DHCP
domain=wlan     # Local wireless DNS domain
address=/gw.wlan/192.168.101.1
                # Alias for this router

# 绑定 Mac 地址
dhcp-host=Mac Address,10.0.1.2
dhcp-host=Mac Address,10.0.1.3
```

还得设置一下固定树莓派ip

```config
#/etc/dhcpcd.conf
interface wlan0
    static ip_address=192.168.101.1/24
    nohook wpa_supplicant
```

## 其他
### 防火墙
原文用的是另外几个软件，[看这里](https://www.raspberrypi.org/documentation/configuration/wireless/access-point-routed.md)

```sh
# iptables
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables-save -f /etc/iptables/iptables.rules
```


### 恢复
重新把无线网卡作为wifi接收器
1. 注释 /etc/dhcpcd.conf 中的固定ip。
2. 关`dnsmasq`和`hostapd`，重启。
```
sudo systemctl disable dnsmasq.service hostapd.service
reboot
```


### [list the connected devices on my wifi access point](https://unix.stackexchange.com/questions/40087/is-there-a-way-to-list-the-connected-devices-on-my-wifi-access-point)
```sh
iw dev wlan0 station dump
sudo arp

cat /var/lib/misc/dnsmasq.leases
```

