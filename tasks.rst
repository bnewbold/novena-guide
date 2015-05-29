
Common Tasks
================

Installing and Running Debian Linux on a SATA Disk
----------------------------------------------------

These instructions will result in a system that boots u-boot and the Linux
kernel from the internal microSD slot, then loads the rootfs and entire
userland operating system from an ext4 partition on a SATA hard disk.

In addition to increased disk capacity, using a SATA disk as a rootfs should
vastly improve disk I/O and thus general system performance.

Connect the SATA disk while the system is powered down, then boot up from the
microSD card. Note that there are two SATA-like connectors on the Novena
mainboard: one is for power from the battery board, and the other is the actual
SATA connection. It isn't possible to connect to the wrong port because the
polarities are flipped. It's strongly recommended to use some form of an
enclosure or mechanical support to prevent the SATA disk from coming detatched
while in use, which could obviously result in data loss.

First make sure required tools are installed::

    sudo apt-get install parted debootstrap

We're going to assume that the SATA disk is blank and unpartitioned, and **all
data on the disk will be overwritten**.

We're going to use the offical ``novena-image.sh`` install script. Checkout out
xobs' novena-image repo::

    sudo apt-get install apt-cacher-ng
    cd
    git clone https://github.com/xobs/novena-image

You'll almost certainly want to install apt-cacher-ng and change the mirror
setting (in sata-install.sh) to ``--mirror
"http://127.0.0.1:3142/http.debian.net/debian"`` (the default is a debian mirror
in Hong Kong). We recommend copying ``sata-install.sh`` to ``local-install.sh``
and making changes there. See the `novena-image.sh documentation
<http://kosagi.com/w/index.php?title=Novena_Image_script>`_ for more details.

To configure the kosagi repositories by default, you can download the
kosagi-repo debian package and kosagi signing key with::

    # WARNING: this isn't a secure way to verify the signing key
    gpg --keyserver keyserver.ubuntu.com --recv-keys 03C7B7EC
    gpg --export 03C7B7EC > kosagi.key
    # WARNING: https is not available for this server
    wget http://repo.novena.io/repo/pool/main/k/kosagi-repo/kosagi-repo_1.0-r1_all.deb
    wget http://repo.novena.io/repo/pool/main/n/novena-eeprom/novena-eeprom_2.1-1_armhf.deb
    wget http://repo.novena.io/repo/pool/main/n/novena-firstrun/novena-firstrun_1.4-r1_all.deb

If you are running from the microSD card and have limited space, you'll
probably want to cull down the default installed package list significantly.
Here is an example ``local-install.sh``::

    #!/bin/bash
    if [ -z $1 ]
    then
            echo "Usage: $0 [device]"
            echo "E.g. $0 /dev/sda"
            exit 1
    fi

    echo "Constructing a disk image on $1"
    exec sudo ./novena-image.sh \
            -d $1 \
            -m "http://127.0.0.1:3142/http.debian.net/debian" \
            -t sata \
            -s jessie \
            -k kosagi.key \
            -a kosagi-repo_1.0-r1_all.deb \
            -a novena-eeprom_2.1-1_armhf.deb \
            -a novena-firstrun_1.4-r1_all.deb \
            -l "sudo openssh-server ntp ntpdate \
                vim powermgmt-base i2c-tools"

When ``local-install.sh`` looks good, run the script::

    # WARNING: VERY DANGEROUS COMMAND!
    sudo ./local-install.sh /dev/sda

To actually boot from SATA you must set a flag in EEPROM. Run ``novena-eeprom``
and take note of the Features list, then add ``sataroot`` to the list and write
it::

    # Edit this list for your board
    novena-eeprom -f es8328,pcie,gbit,hdmi,eepromoops,sataroot -w

The ``novena-image.sh`` script should have taken care of setting the SATA
disk's ID correct, so you're ready to reboot!

After booting into this fresh system, you might want to loop back to the
:doc:`quickstart-board` page. A few errata and things that might pop up:

- need to fix ``/etc/apt/sources.list`` to remove localhost prefix (from
  apt-cacher-ng)
- run ``apt-get install -f`` to fix any outstanding apt issues
- need to add ``deb http://repo.novena.io/repo/ jessie main`` to get kosagi
  updates (if kosagi-repo wasn't installed)
- if they aren't already installed, run ``apt-get install novena-eeprom
  kosagi-repo novena-disable-ssp novena-usb-hub`` (etc)
- need to create a user (``adduser``, then ``usermod -G`` to add to sudo group)
- need to install a fuller set of packages if only a subset were installed to
  save disk space earlier

Flashing Factory Image to microSD Card
-------------------------------------------

Pairing a Bluetooth Keyboard
-------------------------------

First you need to install the ``firmware-atheros`` package followed by a reboot::

    sudo apt-get install firmware-atheros

After rebooting, you need to enable bluetooth and pair it with your keyboard::

    bluetoothctl -a
    power on
    scan on

If everything goes correctly, your bluetooth keyboard should be listed::
    
    pair <tab>

Pressing tab will auto-complete for you (if a bluetooth device has been
found). A number will appear on your screen which you need to type on
your keyboard and press enter. Now you can connect to your keyboard and
trust it so that in the future the keyboard will be connected to automatically::

    connect <tab>
    trust <tab>
    default-agent
    quit

Creating a Simple WiFi Access Point (Hotspot)
----------------------------------------------

.. warning::

    Unfortunately, the shipped Novena kernel (3.17.0-rc5-00217-gfd79638) does
    not include netfilter support, which is required for iptables to no
    NAT IPv4 masquerading. The below instructions will only partially work,
    and clients won't have actual internet access.

    Upgrading to the 3.19 linux-novena kernel should resolve this problem, as
    the necessary modules are compiled in by default. You can test by running
    ``sudo iptables -L``; if this returns an error about kmod and insmod, you
    need to upgrade.

The PCIe 802.11 WiFi card that ships with the Novena supports Access Point mode
(AP, also known as 'master' mode), so in addition to connecting to wireless
gateways and routers as a client, Novena can share connections or even act as a
router/gateway itself.

If you aren't using the included PCIe card for WiFi (eg, using a USB dongle),
you'll need to check that your hardware supports AP mode::

    sudo iw list | less
    # must have 'AP' in "Supported interface modes"

The included PCIe card is based on the Atheros AR9462 chipset.

As background, creating a wireless station and allowing clients to connect is
relatively simple. Sharing an upstream internet connection (eg, from the wired
ethernet jacks) is a bit more complicated. There are at least two common
methods to do so. The first is network bridging, where the Novena routes
packets between the two network connections without acting as a gateway in any
other way. In this configuration a pre-existing router would act as a DHCP
server and gateway to the outside network. In the second configuration Novena
would act as a router/gateway itself and do NAT (Network Address Translation).
In this configuration clients would get DHCP and DNS from Novena on a private
subnetwork. The Novena would translate the IP addresses on any packets going to
and from connected clients to the upstream internet.

NetworkManager is an easiest way to create an access point, and it uses the NAT
scheme by default, with dnsmasq and iptables behind the scenes supplying
DNS/DHCP and NAT rewriting respectively. These directions assume you have
``network-manager`` already installed.

.. note::

   These directions describe a simple mechanism for sharing an internet
   connection. This is not intentended to be a way to have the Novena run as a
   secure or robust wireless gateway. In particular, no firewall is in place,
   your Novena may not be very security hardened by default, the default
   settings may not play well with some devices or networks, etc, etc.

If you have a headless (no GUI) system, you can control NetworkManager using
``nmtui``, otherwise you can use the Gnome GUI.

First make sure you have a working wired (ethernet) connection to the internet.
Then create a new shared WiFi connection. It's recommended to give the
connection (distinct from the SSID) a short name like "wlan0-ap" instead of the
default "Wi-Fi connection 1". Select or enter ``wlan0`` as the hardware device.
In WiFi settings choose an SSID and set the Mode to "Access Point". Add WPA2
security if you like. In the IPv4 network section change the configuration from
"Automatic" to "Shared". The other settings can be left as defaults. Make sure
"Automatically connect" is selected. Save and exit.

The connection may come up automatically after a few minutes. Unlike wired
connections, the connection will not show up in the list of available WiFi
connections in the ``nmtui`` "Active a connection" list. You can check
``/var/log/daemon.log`` for status and error messages, or ``nmcli connection``
for a list of active connections. You can force NetworkManager to bring up the
connection with::

    sudo nmcli connection up wlan0-ap
    # where 'wlan0-ap' is the connection name you chose earlier

To shutdown the access point and return Novena to client mode, the easiest
route seems to be disabling the Auto-connect flag in the wlan0-ap settings,
then run ``sudo nmcli connection down wlan0-ap``, wait a minute, then you
should be given a list of access points to connect to as usual.

Compiling and Installing the Kernel
-------------------------------------

Check out the novena kernel tree::

    git clone https://github.com/xobs/novena-linux

Check out the version you want to build. For example::

    cd novena-linux
    git checkout v3.19-novena

Set the default build configuration and compile the kernel::

    make novena_defconfig
    make -j4

Now that the kernel is compiled, we must install it and its
corresponding set of modules. For the time being the kernel
needs to be on the small /boot partition on the sd card::

    sudo make modules_install
    sudo cp arch/arm/boot/dts/imx6q-novena.dtb /boot/novena.dtb
    sudo cp arch/arm/boot/zImage /boot/zimage

If you have trouble booting the new kernel, hold down the user
(square) button during boot. That should select the kernel in
the sd card's recovery partition. If all else fails, reflash
the sd card with a factory image.

Kosagi's latest kernel build is available in their repo as the
'linux-image-novena' package. The 'u-boot-novena' package also
contains a script to maintain the sdcard card partition, so if
this is installed, the traditional debian 'fakeroot make-kpkg'
method will work without the manual copying above.

Using an External HDMI Monitor
-------------------------------------

These instructions are oriented towards users of a bare mainboard system, not
Desktop or Laptop folks trying to use a secondary display.

Attaching an HDMI monitor should Just Work as a console login; you'll need a
USB keyboard or other input device to log in.

Note that when an external monitor is attached at boottime, the kernel boot and
console login stops working on the UART serial connection and is redirected to
the monitor instead.

For a simple XFCE-based desktop with common applications, install::

    sudo apt-get install task-xfce-desktop xorg-novena \
        xserver-xorg-video-armada xserver-xorg-video-armada-etnaviv iceweasel \
        arandr libetnaviv

.. note::
    As of January 2015, there seems to be an issue_ with the novena-xorg
    package that prevents the "armada" driver from working. A workaround is to
    edit the file ``/usr/share/X11/xorg.conf.d/60-novena.conf`` and add the
    following lines to the top::

        Section "Files"
            ModulePath "/usr/lib/xorg/modules/"
            ModulePath "/usr/lib/arm-linux-gnueabihf/xorg/modules/"
        EndSection

    If this does not work, you can also try replacing the ``armada`` driver in
    that file with ``fbdev`` (and comment out the following option lines) to
    use a (slow) raw framebuffer device instead.

.. _issue: https://github.com/xobs/xorg-novena/issues/2

After future reboots, when the external display is attached you should get a
friendly GUI login screen.

To start up X without rebooting, run ``startxfce4`` from the console login.

