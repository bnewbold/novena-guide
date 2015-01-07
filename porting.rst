
Porting Operating Systems
============================

In general, if trying to port things to the Novena, a good place to start would
be finding existing ports to other hardware platforms based on the Freescale
i.MX6 processors. Eg, the Wandboard, the Gateworks Ventana, the Solidrun
CuBox-i4Pro, and Boundary Devices' Nitrogen6.

GNU/Linux Distributions
--------------------------

It should be reasonably easy to port arbitrary GNU/Linux distributions to
Novena. Either the existing microSD boot partition, u-boot, and kernel can be
used (with the distro rootfs on a SATA disk), or an entire fresh image could be
generated. Many of xobs's patches to the Linux kernel have been upstreamed to
mainline, which helps immensely.

Additional work will be necessary on non-debian-based distributions to port the
userland utilities for things like EEPROM access and pulseaudio support. All
these utilities are of course free software, and can be found by poking around
on the Novena wiki (try starting at `Novena packaing overview
<http://kosagi.com/w/index.php?title=Novena_packaging_overview>`_).

Some earlier work had been done on an `OpenEmbedded build
<http://kosagi.com/w/index.php?title=Building_novena_firmware>`_.

Other Operating Systems
--------------------------

Wikipedia `states <https://en.wikipedia.org/wiki/I.MX#Software_solutions>`_
that basic i.MX6 support has been included in FreeBSD, OpenBSD, and Android.
There is probably also vendor (Freescale) support for some proprietary
industrial operating systems like QNX and VxWorks.

Beyond that it will probably be a matter of studying the documentation and
refering to the Linux implementation for pointers.
