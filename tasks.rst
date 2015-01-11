
Common Tasks
================

.. note:: This page is a work-in-progress stub.

Installing and Running Debian Linux on a SATA Disk
----------------------------------------------------

These instructions will result in a system that boots u-boot and the Linux
kernel from the internal microSD slot, then loads the rootfs and entire
userland operating system from an ext4 partition on a SATA hard disk.

Connect the SATA disk while the system is powered down, then boot up from the
microSD card. Note that there are two SATA-like connectors on the Novena
mainboard: one is for power from the battery board, and the other is the actual
SATA connection. It isn't possible to connect to the wrong port because the
polarities are flipped.

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
<http://kosagi.com/w/index.php?title=Novena_Image_script>`_ for more details on
how to, eg, auto-populate the kosagi signing key into the new image.

If you are running from the microSD card and have limited space, you'll
probably want to cull down the default installed package list significantly.

When ``local-install.sh`` looks good, run the script::

    # WARNING: VERY DANGEROUS COMMAND!
    sudo ./local-install.sh /dev/sda

To actually boot from SATA we need to set a flag in EEPROM. Run
``novena-eeprom`` and take note of the Features list, then add ``sataroot`` to
the list and write it::

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
  updates
- if they aren't already installed, run ``apt-get install novena-eeprom
  kosagi-repo novena-disable-ssp novena-usb-hub`` (etc)
- need to create a user (``adduser``, then ``usermod -G`` to add to sudo group)
- need to install a fuller set of packages if only a subset were installed to
  save disk space earlier

Flashing Factory Image to microSD Card
-------------------------------------------

Pairing a Bluetooth Keyboard
-------------------------------

Creating a WiFi Hostspot
---------------------------

Compiling and Installing the Kernel
-------------------------------------
