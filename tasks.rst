
Common Tasks
================

.. note:: This page is a work-in-progress stub.

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
setting (in sata-install.sh) to '--mirror
"http://127.0.0.1:3142/http.debian.net/debian"' (the default is a debian mirror
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

- need to fix /etc/apt/sources.list to remove localhost prefix (from
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

Creating a WiFi Hostspot
---------------------------

Compiling and Installing the Kernel
-------------------------------------

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

