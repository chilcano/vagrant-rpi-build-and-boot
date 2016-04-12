# Vagrant box for Raspberry Pi (arm) cross-compiling using Vagrant, VirtualBox, Ansible and Python

![Provisioning massively cross-compiled binaries to Raspberry Pi (armv7) using Vagrant, VirtualBox, Ansible and Python](https://github.com/chilcano/vagrant-rpi-build-and-boot/blob/master/blog-cross-compiling-kismet-raspberrypi-arm.png "Provisioning massively cross-compiled binaries to Raspberry Pi (armv7) using Vagrant, VirtualBox, Ansible and Python")

## Requirements:

I'm using a Mac OS X (El Capitan - Version 10.11.3) with the next tools:

- VirtualBox 5.0.16
- Vagrant 1.8.1
- Ansible 2.0.1.0 (installed via Pip)
- Python 2.7.11
- Raspberry Pi 2 Model B 
- Raspbian OS (2015-09-24-raspbian-jessie.img)
- OpenFramework v0.9.0 for cross-compiling (http://openframeworks.cc/versions/v0.9.0/of_v0.9.0_linuxarmv7l_release.tar.gz)


## Preparing the Raspberry Pi


__1. Copy RPi image to SD__

Identify the disk (not partition) of your SD card, unmount and copy the image there:
```bash
$ diskutil list
$ diskutil unmountDisk /dev/disk2
$ cd /Users/Chilcano/Downloads/@isos_vms/raspberrypi-imgs
$ sudo dd bs=1m if=2015-09-24-raspbian-jessie.img of=/dev/rdisk2
```

__2. Connect the Raspberry Pi directly to your Host (MAC OS X)__

Using an ethernet cable, connect your Raspberry Pi to your Host, in my case I've a MAC OS X and I'm going to share my WIFI Network connection. 
Then, enabling `Internet Sharing` and the "Thunderbolt Ethernet" an IP address will be assigned to the Raspberry Pi, also Raspberry Pi will have Internet access/Network access and the MAC OS X can connect via SSH to the Raspberry Pi. 
All that will be possible without a hub, switch, router, screen or keyboard, etc. This will be useful, because we are going to install new software in Raspberry Pi.

After connect your Raspberry Pi to your MAC OS X, turn on by connecting an USB cable, in your MAC OS X open a Terminal and issue a SSH command, before re-generate the SSH keys.

Note that the default hostname of any Raspberry Pi is `raspberrypi.local`.

```bash
// cleaning existing keys
$ ssh-keygen -R raspberrypi.local

// connect to RPi using `raspberry` as default password
$ ssh pi@raspberrypi.local
```

After connecting, you will check the assigned IP address and the shared Internet Connection. Now, check out your connection.
```bash
pi@raspberrypi:~ $ ping www.docker.com
```

__3. Configure your RPi__

Boot your RPi and open a shell. Then enter:
```bash
pi@raspberrypi:~ $ sudo raspi-config
```
In the `raspi-config` menu, select `Option 1 Expand Filesystem`, change Keyboard layout, etc. and reboot.

Just if `mirrordirector.raspbian.org` mirror is not available, remove `http://mirrordirector.raspbian.org/raspbian/` repository and add a newest.
```bash
pi@raspberrypi ~ $ sudo nano /etc/apt/sources.list

#deb http://mirrordirector.raspbian.org/raspbian/ jessie main contrib non-free rpi
deb http://ftp.cica.es/mirrors/Linux/raspbian/raspbian/ jessie main contrib non-free rpi

# Uncomment line below then 'apt-get update' to enable 'apt-get source'
#deb-src http://archive.raspbian.org/raspbian/ jessie main contrib non-free rpi
```

__4. Install OpenFrameworks tools and dependencies into Raspberry Pi__

Download and unzip OpenFrameworks into RPi under `/opt`.

```bash
pi@raspberrypi:~ $ cd /opt
pi@raspberrypi:/opt $ sudo wget http://openframeworks.cc/versions/v0.9.0/of_v0.9.0_linuxarmv7l_release.tar.gz
pi@raspberrypi:/opt $ sudo tar -zxf of_v0.9.0_linuxarmv7l_release.tar.gz
pi@raspberrypi:/opt $ sudo rm of_v0.9.0_linuxarmv7l_release.tar.gz
```

Now, update the dependencies required when cross-compiling by running `install_dependencies.sh`.
```bash
pi@raspberrypi:~ $ sudo /opt/of_v0.9.0_linuxarmv7l_release/scripts/linux/debian/install_dependencies.sh
```

Now, compile oF, compile and execute an oF example.
```bash
// compiling oF
pi@raspberrypi:~ $ sudo make Release -C /opt/of_v0.9.0_linuxarmv7l_release/libs/openFrameworksCompiled/project
...
se/libs/openFrameworksCompiled/lib/linuxarmv7l/obj/Release/libs/openFrameworks/math/ofMatrix4x4.o /opt/of_v0.9.0_linuxarmv7l_release/libs/openFrameworksCompiled/lib/linuxarmv7l/obj/Release/libs/openFrameworks/math/ofQuaternion.o /opt/of_v0.9.0_linuxarmv7l_release/libs/openFrameworksCompiled/lib/linuxarmv7l/obj/Release/libs/openFrameworks/math/ofVec2f.o
HOST_OS=Linux
HOST_ARCH=armv7l
checking pkg-config libraries:   cairo zlib gstreamer-app-1.0 gstreamer-1.0 gstreamer-video-1.0 gstreamer-base-1.0 libudev freetype2 fontconfig sndfile openal openssl libpulse-simple alsa gtk+-3.0
Done!
make: Leaving directory '/opt/of_v0.9.0_linuxarmv7l_release/libs/openFrameworksCompiled/project'

// executing an example
pi@raspberrypi:~ $ sudo make -C /opt/of_v0.9.0_linuxarmv7l_release/apps/myApps/emptyExample
pi@raspberrypi:~ $ cd /opt/of_v0.9.0_linuxarmv7l_release/apps/myApps/emptyExample
pi@raspberrypi /opt/of_v0.9.0_linuxarmv7l_release/apps/myApps/emptyExample $ bin/emptyExample
```

__5. Make an new image file from the existing and updated Raspberry Pi__

Remove the SD card from the Raspberry Pi, insert the  SD card in your Host (in my case is MAC OS X) and use `dd` to make an new image file.
```bash
$ diskutil list
$ diskutil unmountDisk /dev/disk2
$ sudo dd bs=1m if=/dev/rdisk2 of=2015-09-24-raspbian-jessie-of2.img

15279+0 records in
15279+0 records out
16021192704 bytes transferred in 381.968084 secs (41943799 bytes/sec)
```

_Very important_:

- The `2015-09-24-raspbian-jessie-of.img` will be `shared` and after `mounted` from the guest VM, for that, set the user and permissions to `2015-09-24-raspbian-jessie-of.img` as shown below:

```bash
$ sudo chmod +x 2015-09-24-raspbian-jessie-of2.img
$ sudo chown Chilcano 2015-09-24-raspbian-jessie-of2.img

$ ls -la
total 110439056
drwxr-xr-x  33 Chilcano  staff         1122 Apr 11 19:12 ./
drwxr-xr-x  35 Chilcano  staff         1190 Mar 23 19:26 ../
-rwxr-xr-x   1 Chilcano  staff  16021192704 Apr 11 19:19 2015-09-24-raspbian-jessie-of2.img*
-rwxr-xr-x   1 Chilcano  staff   4325376000 Apr 11 17:02 2015-09-24-raspbian-jessie.img*
...
```


## Building the Vagrant box

__1. In your MAC OS X, to clone the `rpi-build-and-boot` github repository__

```bash
$ git clone https://github.com/twobitcircus/rpi-build-and-boot
$ cd rpi-build-and-boot
```

Copy/Move the newest RPi image created above into `rpi-build-and-boot` folder.
```bash
$ mv /Users/Chilcano/Downloads/@isos_vms/raspberrypi-imgs/2015-09-24-raspbian-jessie-of2.img .
```

__2. Install Vagrant and vbguest plugin into MAC OS X__

```bash
$ wget https://releases.hashicorp.com/vagrant/1.8.1/vagrant_1.8.1.dmg
$ vagrant plugin install vagrant-vbguest
```

__3. Create a new `Vagrantfile` with VirtualBox as provider in the same folder `rpi-build-and-boot`__


Here the `Vagrantfile` (https://github.com/chilcano/vagrant-rpi-build-and-boot/blob/master/Vagrantfile) for VirtualBox.


__4. Getting `boot` and `root` partitions offsets to do loop mounting in Vagrant__

Using `./tool.py offsets <my_image.img>` I will get the offsets of the `boot` and `root` partitions, after getting offset, copy the output of this tool to the top of `playbook.yml`. 
To run `tool.py` in MAC OS X, you will need `Python` configured.

```bash
$ ./tool.py offsets 2015-09-24-raspbian-jessie-of2.img

    image: 2015-09-24-raspbian-jessie-of2.img
    offset_boot: 4194304
    offset_root: 62914560
```

The idea to loop-mount the RPi image is to create a full structure of directories and files of a Raspberry Pi distribution under a mounting-point in a Vagrant box. This structure is required to do `cross-compiling` and move/copy new binaries and ARM cross-compiled binaries.


__5. Mounting Raspberry Pi image and booting from Vagrant using NFS__

Using `./tool.py netboot image.img /dev/rdiskX [--ip=10.0.0.Y]` you will copy just the `boot` partition in a new and tiny SD card. 
This new SD card with a fresh `boot` partition will be useful to boot from the network/remotely. The RPi will download the `root` partition from Vagrant, in fact, Vagrant will be sharing the custom RPi image (`2015-09-24-raspbian-jessie-of2.img`) via NFS to any Raspberry Pi connected to same network and having a pre-loaded `boot` partition.

The idea behind is to provision a custom RPi image massively avoiding to waste time copying and creating SD card for each Raspberry Pi. Also, this method is useful to provision software, configuration, packages, or in my case, provide cross-compiled software for ARM architectures massively.

```bash
$ diskutil list

// a new SD on disk3 will be used
$ diskutil unmountDisk /dev/disk3

$ ./tool.py netboot 2015-09-24-raspbian-jessie-of2.img /dev/rdisk3

```

Note that `tool.py netboot` automatically will assigns to RPi the `10.0.0.101` as IP address and `8.8.8.8` and `8.8.4.4` as DNS servers to `eth0`.
You can check or modify previously these values by editing the `cmdline.txt` file placed in the `boot` RPi partition. You can edit it from a running Raspberry Pi or from a mounted partition.


__6. Download and unzip oF (OpenFramework) into `rpi-build-and-boot` folder__

If you forgot copy OpenFramework in your RPi, you can do now. Using the Ansible `playbook.yml`, the `oF` will be copied to your RPi.
```bash
$ cd rpi-build-and-boot
$ wget http://openframeworks.cc/versions/v0.9.0/of_v0.9.0_linuxarmv7l_release.tar.gz
$ tar -zxf of_v0.9.0_linuxarmv7l_release.tar.gz
```


__7. Update the Ansible `playbook.yml`__ 

I've had to tweak the `playbook.yml` to avoid warnings, add DNS to `cmdline.txt` and add `iptables` filters to get Internet access on RPi using Host shared NIC. 

Here the updated Ansible `playbook.yml` (https://github.com/chilcano/vagrant-rpi-build-and-boot/blob/master/playbook.yml).


__8. Create the Vagrant box__

```bash
$ cd rpi-build-and-boot
$ vagrant up --provider virtualbox
```

... let's have coffee  ;)

After that, restart the Vagrant box recently created.

```bash
$ vagrant halt
$ vagrant up
```

Connect your Raspberry Pi -with the SD card and boot partition copied- using ethernet clable to your Host PC (in my case is a Mac OS X), wait some seconds and check if Raspberry Pi has started from the `root` partition shared by NFS from the Vagrant box.

```bash
$ ping raspberrypi.local
$ ping 10.0.0.101
```

And check if Raspberry Pi is running but from Vagrant box.
```bash
$ vagrant ssh

vagrant@vagrant-ubuntu-trusty-64:~$ ifconfig
eth0      Link encap:Ethernet  HWaddr 08:00:27:c9:24:d6
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fec9:24d6/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:665 errors:0 dropped:0 overruns:0 frame:0
          TX packets:427 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:67162 (67.1 KB)  TX bytes:54225 (54.2 KB)

eth1      Link encap:Ethernet  HWaddr 08:00:27:b3:e9:a4
          inet addr:10.0.0.1  Bcast:10.0.0.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:feb3:e9a4/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:29474 errors:0 dropped:0 overruns:0 frame:0
          TX packets:60947 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:5247033 (5.2 MB)  TX bytes:70887820 (70.8 MB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

vagrant@vagrant-ubuntu-trusty-64:~$ ping 10.0.0.101

vagrant@vagrant-ubuntu-trusty-64:~$ ping google.com
```


__9. Check if ARM cross-compiling works in the VirtualBox guest__

Check if the cross-compiling Variables have been defined.
```bash
vagrant@vagrant-ubuntu-trusty-64:~$ cat /home/vagrant/.profile

...
export GST_VERSION=1.0
export RPI_ROOT=/opt/raspberrypi/root
export TOOLCHAIN_ROOT=/opt/cross/bin
export PLATFORM_OS=Linux
export PLATFORM_ARCH=armv7l
export PKG_CONFIG_PATH=$RPI_ROOT/usr/lib/arm-linux-gnueabihf/pkgconfig:$RPI_ROOT/usr/share/pkgconfig:$RPI_ROOT/usr/lib/pkgconfig
```

Check if RPi has been mounted.
```bash
vagrant@vagrant-ubuntu-trusty-64:~$ ll /opt/raspberrypi/boot/
vagrant@vagrant-ubuntu-trusty-64:~$ ll /opt/raspberrypi/root/
```

And check if oF works by compiling an example.
```bash
$ make -C /opt/openframeworks/apps/myApps/emptyExample
```