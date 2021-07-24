# PINEPHONE
You can find here my personal experiences with [pinephone][1], my first linux phone.

### Initial Set-Up

It all started at Christmas Eve, thanks to my awesome wife, when I got my first linux phone [**PINEPHONE**][1]. The phone was preloaded with manjaro OS e.g. [manjaro community edition][2] and assembled with 3GB RAM, 32GB eMMC storage.

**Power UP**

To power up the phone you need to remove platic sticker which isolates battery from the phone. You can find detailed instructions in [pinephone wiki][3], section _first time installation_. Once done I booted the phone into OS by pressing the power button, bigger one on the right side. Suprisingly it worked without issues :)
Though I encountered the first issue a second later.

**Black screen after login**.

Manjaro OS default user is called _manjaro_ with password _123456_. You need to login with password everytime you use the phone. Once I entered the password the screen became black. The issue repeated also after reboot. Fortunately I found [this][4] closed issue reported on manjaro. It seems to caused by proxmity sensor and it is considered as HW issue. The workaround is to blacklist kernel module for the sensor.

- I wasn't able to access OS therefore I had to download & flash _jumpdrive_ utility to sdcard as mentioned in the [pinephone installation istruction wiki][5], section, _Installation to the eMMC_. It allow you to boot from micro SD and exposes the internal eMMC flash storage. You can used it as another device when the PinePhone is connected to a computer.

- Mount eMMC storage and blacklist proximity sensor.

```
$ mkdir /mnt/usb
# replace 'partition_id' with real partition identifier e.g. sda2
$ sudo mount /dev/partition_id /mnt/usb/
$ echo 'blacklist stk3310' |sudo tee -a /mnt/usb/etc/modprobe.d/proximity.con
```

- Diconnect the phone from PC, remove SD card and re-boot it into OS. Now you should log in successfully.

So far I was impressed and excited that it really works. I was able to connect to wifi and use firefox to browse the internet. Terminal is cute :) BUT not everything is ideal. There is still bunch of work to make user experience "great" because it is sometimes laggy, application rendering is sometimes off, etc.

**Phone Update**

After a while I decided that it was good time for the first system update. Initially I tried to update it via UI: settings -> about -> software updates. It was loading updates for several minutes therefore I decided to try more _linux_ approach e.g. leverage terminal. So far I'm not familiar with [_packman_][8] package manager used on arch based OSes but I found useful write up [pinephone tips & tricks][6] which help me to get it done.

- Update list. You don't have to use any flags for [pacman-s][7] tool ... but I decided to use these:
  - _--continent_ create a custom mirror pool from countries within the geolocated continent.
  - _--api --protocol https_ use only mirrors available via HTTPs protocol.

```
sudo pacman-mirrors --continent --api --protocol https
```

- Update packages.

```
sudo pacman -Syyu
```

I had to reboot the phone after update because I wasn't able to log in.

### Encountered Issues

#### locale cannot be set - [link][9].

_phosh_ service has backed locale, _LANG_ environment variable in systemd unit file.
It affects all services started via UI because they are started by _phosh_.
Therefore they inherit _LANG_ variable.
To hot-fix it we need to remove a line which defines _LANG_ variable in phosh unit file.
Phosh unit file is stored in the path _/etc/systemd/system/phosh.service_.

You can run following _sed_ comand which does the job for you e.g. it comment the line with LANG variable:

```
sed -r -i 's/^(Environment=LANG=.*)$/#\1/' /etc/systemd/system/phosh.service
```

Afterwards we need to reload systemd configuration files & restart phosh service:

```
sudo systemctl daemon-reload && sudo systemctl restart phosh
```

#### Phone isn't discoverable via bluetooth - [link][10].
#### Login errors - [link][11].
#### Terminal is reporting missing configuration - [link][12].
#### errors reported by pacman during upgrade to testing branch - [link][13].
#### Journal led errors - [link][14].
#### Phone drops off wifi when sleeping and on battery - [link][15]

User session is suspended due to inactivity when the threshold is fulfilled and phone is powered from battery.
Once the session is suspended it stops wifi and modem processes.
This can be seen in journal logs:

```
Jan 01 14:36:46 manjaro-arm NetworkManager[3640]: <info>  [1609508206.6183] manager: sleep: sleep requested (sleeping: no  enabled: yes)
Jan 01 14:36:46 manjaro-arm NetworkManager[3640]: <info>  [1609508206.6188] device (p2p-dev-wlan0): state change: disconnected -> unmanaged (reason 'sleeping', sys-iface-state: 'managed')
Jan 01 14:36:46 manjaro-arm ModemManager[3737]: <info>  [sleep-monitor] system is about to suspend
Jan 01 14:36:46 manjaro-arm NetworkManager[3640]: <info>  [1609508206.6219] manager: NetworkManager state is now ASLEEP
Jan 01 14:36:46 manjaro-arm NetworkManager[3640]: <info>  [1609508206.6235] device (wlan0): state change: activated -> deactivating (reason 'sleeping', sys-iface-state: 'managed')
Jan 01 14:36:46 manjaro-arm dbus-daemon[3639]: [system] Activating via systemd: service name='org.freedesktop.nm_dispatcher' unit='dbus-org.freedesktop.nm-dispatcher.service' requested by ':1.8' (uid=0 pid=3640 comm="/usr/bin/Net>
Jan 01 14:36:46 manjaro-arm systemd[1]: Starting Network Manager Script Dispatcher Service...
Jan 01 14:36:46 manjaro-arm dbus-daemon[3639]: [system] Successfully activated service 'org.freedesktop.nm_dispatcher'
Jan 01 14:36:46 manjaro-arm systemd[1]: Started Network Manager Script Dispatcher Service.
Jan 01 14:36:46 manjaro-arm wpa_supplicant[3772]: wlan0: CTRL-EVENT-DISCONNECTED bssid=74:83:c2:91:c8:d6 reason=3 locally_generated=1
Jan 01 14:36:46 manjaro-arm systemd-networkd[3369]: wlan0: Lost carrier
Jan 01 14:36:46 manjaro-arm systemd-timesyncd[3634]: No network connectivity, watching for changes.
Jan 01 14:36:46 manjaro-arm wpa_supplicant[3772]: wlan0: CTRL-EVENT-REGDOM-CHANGE init=CORE type=WORLD
Jan 01 14:36:46 manjaro-arm NetworkManager[3640]: <info>  [1609508206.7873] device (wlan0): supplicant interface state: completed -> disconnected
Jan 01 14:36:46 manjaro-arm NetworkManager[3640]: <info>  [1609508206.7883] device (wlan0): state change: deactivating -> disconnected (reason 'sleeping', sys-iface-state: 'managed')
Jan 01 14:36:46 manjaro-arm NetworkManager[3640]: <info>  [1609508206.8191] dhcp4 (wlan0): canceled DHCP transaction
Jan 01 14:36:46 manjaro-arm NetworkManager[3640]: <info>  [1609508206.8193] dhcp4 (wlan0): state changed bound -> done
Jan 01 14:36:46 manjaro-arm NetworkManager[3640]: <info>  [1609508206.8469] device (wlan0): state change: disconnected -> unmanaged (reason 'sleeping', sys-iface-state: 'managed')
Jan 01 14:36:46 manjaro-arm systemd-networkd[3369]: wlan0: Link DOWN
Jan 01 14:36:47 manjaro-arm systemd[1]: Reached target Sleep.
Jan 01 14:36:47 manjaro-arm systemd[1]: Starting Suspend...
Jan 01 14:36:47 manjaro-arm systemd-sleep[5886]: Suspending system...
Jan 01 14:36:47 manjaro-arm kernel: PM: suspend entry (deep)
Jan 01 14:36:47 manjaro-arm wpa_supplicant[3772]: nl80211: deinit ifname=wlan0 disabled_11b_rates=0
```

Default gnome settings:
- inactivity threshold is set to 5 minutes when powered from battery:

```
$ dbus-launch gsettings get org.gnome.settings-daemon.plugins.power sleep-inactive-battery-timeout
300
```

- the phone is suspended when the threshold is fulfilled:

```
$ dbus-launch gsettings get org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type
'suspend'
```

Inactivity type needs to be adjusted if you don't want to stop / kill wifi due to inactivity.
All possible types which you can use:

```
$ dbus-launch gsettings range org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type
enum
'blank'
'suspend'
'shutdown'
'hibernate'
'interactive'
'nothing'
'logout'
```


The most suitable option seems to be _blank_. It turns off the screen when the user is inactive.

```
dbus-launch gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type blank
```

I use these options so far:

```
dbus-launch gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-timeout 60
dbus-launch gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-timeout 300
dbus-launch gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type blank
dbus-launch gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type blank
```

Furthermore you can set these aliases in `~/.bashrc` so you can switch battery type based on your need:

```
alias sleep_inactive_type_blank='dbus-launch gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type blank'
alias sleep_inactive_type_suspend='dbus-launch gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type suspend'
```

You can find more information about gnome settings on these links:
- https://people.gnome.org/~pmkovar/system-admin-guide/automatic-logout.html
- https://blog.sleeplessbeastie.eu/2020/08/19/how-to-alter-ubuntu-desktop-configuration-using-terminal/

#### Failed to boot after upgrade.

Most recently I ran [an update](#phone_update), on ~20.07, which prevented me to log into OS.
I logged into PH via SSH and run [update command](#phone_update). The updates run several minutes.
I saw some wierd messages in the terminal about password at the end of the update.
I didn't pay any attention to it because I was doing other stuff around and that was mistake :)
Therefore I just rebooted it. After reboot it dropped me into terminal instead of OS.
There was login prompt asking for password. I didn't have proper keyboard around me to type the password.

<ins>How did I fixed ?</ins>
- I created [jump-drive](5) micro SD card and boot PH into it.
- I mounted main partition which contains **/etc**.
- I set an empty password for my user in **/etc/passwd** by removing the x after the user name.
- I powered of PH, remove SD card and power it on.
- PH booted normaly into OS. I was able to log in via UI with old password but logging via SSH didn't work.
- I changed my password in the terminal with passwd command and SSH login start to work as well.

<ins>Additional notes:</ins>
- I don't really know why it broke and I don't really have time to investigate it properly.
- I use testing branch instead of stable one.

[1]: https://www.pine64.org/pinephone/
[2]: https://www.pine64.org/2020/08/31/pinephone-manjaro-community-edition/
[3]: https://wiki.pine64.org/wiki/PinePhone
[4]: https://gitlab.manjaro.org/manjaro-arm/issues/pinephone/phosh/-/issues/10
[5]: https://github.com/dreemurrs-embedded/Jumpdrive/releases
[5]: https://wiki.pine64.org/wiki/PinePhone_Installation_Instructions
[6]: https://forum.manjaro.org/t/pinephone-tips-and-tricks-experience-and-lessons-learned/39655
[7]: https://wiki.manjaro.org/index.php/Pacman-mirrors
[8]: https://wiki.manjaro.org/index.php/Pacman_Overview
[9]: https://gitlab.manjaro.org/manjaro-arm/issues/pinephone/phosh/-/issues/56
[10]: https://gitlab.manjaro.org/manjaro-arm/issues/pinephone/phosh/-/issues/122
[11]: https://gitlab.manjaro.org/manjaro-arm/issues/pinephone/phosh/-/issues/123
[12]: https://gitlab.manjaro.org/manjaro-arm/issues/pinephone/phosh/-/issues/124
[13]: https://gitlab.manjaro.org/manjaro-arm/issues/pinephone/phosh/-/issues/125
[14]: https://gitlab.manjaro.org/manjaro-arm/issues/pinephone/phosh/-/issues/128
[15]: https://gitlab.manjaro.org/manjaro-arm/issues/pinephone/phosh/-/issues/54
