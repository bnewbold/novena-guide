FPGA Hacking
===================

.. TODO::
    - HDL template
    - kernel and userland details
    - performance and "numbers" table
    - expansion header pinouts

The general purpose FPGA on the Novena mainboard is a Xilinx Spartan6 XC6SLX45
device in the CSG324 BGA package with a ``-3`` speedgrade. It is connected to
the ARM CPU via SPI for bitstream configuration, via i2c for low-speed
communications, and via a 16-bit memory interface ("EIM") for faster
communications. The FPGA also has its own DRAM chip and an expansion header to
which extra hardware can be attached.

.. list-table:: Novena FPGA Specifications (from Xilinx)
    :stub-columns: 1

    * - Part Number
      - Xilinx Spartan6 XC6SLX45
    * - Slices
      - 6822
    * - Logic Cells
      - 43661
    * - CLB Flip-Flops
      - 54576
    * - Max Distributed RAM
      - 50 KB (401 Kb)
    * - Block RAM
      - 261 KB (116x 18 Kb blocks)
    * - Clock Management Tiles (CMT)
      - 4
    * - DSP48A Slices
      - 58
    * - PCIe Hard Blocks
      - None
    * - Memory Controller Blocks
      - 2
    * - Gigabit Transcievers (GTP)
      - None
    * - Bitfile Size
      - 1.49 MB (11.9 Mb)
    * - US Market Price
      - $53 (@100, December 2014)

For those just getting started with FPGA development, two sources of free/libre
learning resources are the `free range factory
<http://www.freerangefactory.org/site/index.php>`_ website (including the libre
"Free Range VHDL" book, and a mirror of opencores.org), and Xess Corp's free
"FPGAs? Now What?!" `book (pdf) <http://www.xess.com/static/media/appnotes/FpgasNowWhatBook.pdf>`_. These guides cover a lot of context about getting
vendor software installed, HDL languages, FPGA hardware resources, timing and
constraint specification, the compilation and synthesis process, and the
various toolchain workflows; this content won't be repeated here.

GPBB FPGA Communications Quickstart
-------------------------------------

The General Purpose Breakout Board (GPBB) comes installed on the expansion
port header of all Novenas. It's got some LEDs and analog I/O (see `the wiki
<http://kosagi.com/w/index.php?title=GPBB_User_Guide>`_ for details), which
make it great for simple FPGA experimentation and blinky light displays.

Before we can control the GPBB from linux, we need to load a bitfile to the
FPGA (which controls the expansion header) and install some userland software
to talk to the FPGA over the EIM interface.

Run these commands from a Novena with the GPBB attached to compile the userland
programs::

    sudo apt-get install i2c-tools libi2c-dev
    cd      # sic.
    git clone https://github.com/bunnie/novena-gpbb-example
    cd novena-gpbb-example
    make -j4

Then use the helper script and bitfile included in the ``novena-gpbb-example``
repo to configure the FPGA (this would need to be done after every power cycle
of the board)::

    sudo ./configure.sh novena_fpga.bit

And check if we can read metadata from the hardware::

    sudo ./novena-gpbb -v

Finally, make with the blinkenlights_ and toggle the four LEDs on the GPBB,
connected to I/O port B, pins 0 to 3::

    sudo ./novena-gpbb -oeb 1 # set port B to drive
    for BIT in 0 1 2 3; do
        sudo ./novena-gpbb -p_set b $BIT # Set pin
        sleep 1
    done
    sudo ./novena-gpbb -p b 00 # Write 0x00, i.e. reset all pins
    sudo ./novena-gpbb -oeb 0 # return port B to tristate mode

Joy!

.. _blinkenlights: https://en.wikipedia.org/wiki/Blinkenlights
