# PINEPHONE
You can find here my personal experiences with [pinephone][1], my first linux phone.

### Initial Set-Up

It all started at Christmas Eve, thanks to my great wife, when I got my first linux phone [**PINEPHONE**][1]. The phone was preloaded with manjaro OS e.g. [manjaro community edition][2].

**Power UP**

To power up the phone you need to remove platic sticker which isolates battery from the phone. You can find detailed instructions in [pinephone wiki][3], section 'first time installation'. Once done I booted the phone into OS by pressing the power button, bigger one on the right side. Suprisingly it worked without issues :)
Though I encountered the first issue a second later.

**Black screen after login**.

Manjaro OS default user is called _manjaro_ with password _123456_. You need to type password before you can use the phone. Once I entered the password the screen became black. The issue repeated also after reboot. Fortunately I found [this][4] closed issue about the similar error. It seems to be proxmity sensor hardware issue. The workaround is to black list kernel module for the sensor.

- I wasn't able to access OS therefore I had to download & flash _jumpdrive_ utility to sdcard as mentioned in the [pinephone installation istruction wiki][5], section, _Installation to the eMMC_. It allow you to boot from micro SD and exposes the internal eMMC flash storage. You can used it as another device when the PinePhone is connected to a computer.

- Mount eMMC storage and blacklist proximity sensor.

```
$ mkdir /mnt/usb
# replace 'partition_id' with real partition identifier e.g. sda2
$ sudo mount /dev/partition_id /mnt/usb/
$ echo 'blacklist stk3310' >> /mnt/usb/etc/modprobe.d/proximity.conf
```

- Diconnect the phone from PC, remove SD card and re-boot it into OS. Now you should log in successfully.

So far I was impressed and excited that it really works. I was able to connect to wifi and use firefox to browse the internet. Terminal is cute :) BUT not everything is ideal. There is still bunch of work to make user experience "great" because it is sometimes laggy, application rendering is sometimes off, etc.

**Phone Update**

After a while I decided that it was good time for the first system update. Initially I tried to update it via UI: settings -> about -> software updates. It was loading several minutes there I was looking for more _linux_ approach e.g. leverage terminal. So far I'm not familiar with _packman_ package manager used on arch based OS but I found useful write up [pinephone tips and tricks] which help me to get it done.

- Update mirror list. You don't have to use any flag for [pacman-mirrors][7] tool ... flags:
  - _--fasttrack 5_ uses mirrors which are up-to-date for your branch. Limit number of mirrors to 5.
  - _--api --protocol https_ use only mirrors available via HTTPs protocol.

```
sudo pacman-mirrors --fasttrack 5 --api --protocol https
```

- Update packages.

```
sudo pacman -Syyu
```

I had to reboot the phone after package update because I wasn't able to log in.

[1]: https://www.pine64.org/pinephone/
[2]: https://www.pine64.org/2020/08/31/pinephone-manjaro-community-edition/
[3]: https://wiki.pine64.org/wiki/PinePhone
[4]: https://gitlab.manjaro.org/manjaro-arm/issues/pinephone/phosh/-/issues/10
[5]: https://github.com/dreemurrs-embedded/Jumpdrive/releases
[5]: https://wiki.pine64.org/wiki/PinePhone_Installation_Instructions
[6]: https://forum.manjaro.org/t/pinephone-tips-and-tricks-experience-and-lessons-learned/39655
[7]: https://wiki.manjaro.org/index.php/Pacman-mirrors
