---
{"dg-publish":true,"permalink":"/__zettel/202503092037NetworkManager更换iwd后端/","title":202503092037,"tags":["networkmanager","iwd","wpa_supplicant"],"created":"2025-03-09T20:37:11+08:00"}
---


此前NetworkManager一直使用wpa_supplicant作为后端，但是`journalctl -f`发现狂刷日志，

```
journalctl -f
Feb 28 10:23:31 baumgard wpa_supplicant[982]: wlan0: CTRL-EVENT-SIGNAL-CHANGE above=0 signal=-73 noise=9999 txrate=175600
Feb 28 10:23:31 baumgard wpa_supplicant[982]: wlan0: CTRL-EVENT-SIGNAL-CHANGE above=0 signal=-73 noise=9999 txrate=175600
Feb 28 10:23:32 baumgard wpa_supplicant[982]: wlan0: CTRL-EVENT-SIGNAL-CHANGE above=0 signal=-73 noise=9999 txrate=175600
Feb 28 10:23:32 baumgard wpa_supplicant[982]: wlan0: CTRL-EVENT-SIGNAL-CHANGE above=0 signal=-70 noise=9999 txrate=175600
Feb 28 10:23:33 baumgard wpa_supplicant[982]: wlan0: CTRL-EVENT-SIGNAL-CHANGE above=0 signal=-70 noise=9999 txrate=175600
Feb 28 10:23:33 baumgard wpa_supplicant[982]: wlan0: CTRL-EVENT-SIGNAL-CHANGE above=0 signal=-70 noise=9999 txrate=6000
Feb 28 10:23:34 baumgard wpa_supplicant[982]: wlan0: CTRL-EVENT-SIGNAL-CHANGE above=0 signal=-71 noise=9999 txrate=6000
Feb 28 10:23:34 baumgard wpa_supplicant[982]: wlan0: CTRL-EVENT-SIGNAL-CHANGE above=0 signal=-70 noise=9999 txrate=175600
Feb 28 10:23:35 baumgard wpa_supplicant[982]: wlan0: CTRL-EVENT-SIGNAL-CHANGE above=0 signal=-70 noise=9999 txrate=175600
Feb 28 10:23:35 baumgard wpa_supplicant[982]: wlan0: CTRL-EVENT-SIGNAL-CHANGE above=0 signal=-70 noise=9999 txrate=6000
```

而且官方LiveCD里面也已经切换到iwd，所以我们也换成iwd作为wifi后端。

更换方法参考：https://forum.endeavouros.com/t/journalctl-wlan0-ctrl-event-signal-change/68475/4
Install iwd

`sudo pacman -S iwd`

create a config file for KDE network manager

`sudo vi /etc/NetworkManager/conf.d/iwd.conf`

addd this lines

`[device] wifi.backend=iwd`

deactivate the wpa_supplicant

`sudo systemctl stop wpa_supplicant.service sudo systemctl disable wpa_supplicant.service`

activate iwd

`sudo systemctl enable iwd.service sudo systemctl start iwd.service`

restart network manager

`sudo systemctl restart NetworkManager`

after that i must reconnect to my wifi with password

now i have a clean journal !