FPGA Hacking
===================

- HDL template
- pointers to toolchain setup and FPGA learning resources
- kernel and userland details
- performance and "numbers" table
- expansion header pinouts

GPBB FPGA Communications Quickstart
-------------------------------------

Run these commands from a Novena with the GPBB attached::

    sudo apt-get install i2c-tools libi2c-dev
    cd      # sic.
    git clone https://github.com/bunnie/novena-gpbb-example
    cd novena-gpbb-example
    make -j4
    sudo ./configure.sh novena_fpga
    sudo ./novena-gpbb -v

