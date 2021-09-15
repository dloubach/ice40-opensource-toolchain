# iCE40 opensource toolchain
This is a compilation of various sources to create a "how to" build a toolchain environment based on open source using **Linux/Ubuntu 20.04 LTS** distro.

This step-by-step guideline aims to build an open source toolchain for iCE40 series FPGA including:
- IceStorm Tools
- Arachne-PNR
- NextPNR
- Yosys
- RISC-V toolchain 

This is totally based on the [James Devine's Blog](https://pingu98.wordpress.com/2019/04/08/how-to-build-your-own-cpu-from-scratch-inside-an-fpga/) and on [Project IceStorm Clifford](http://www.clifford.at/icestorm/).


## FTDI drivers
Get the FTDI driver from [FTDI chip website](https://ftdichip.com/drivers/d2xx-drivers/), or just follow the next command line block:

```bash
wget https://www.ftdichip.com/Drivers/D2XX/Linux/libftd2xx-x86_64-1.4.8.gz
tar xfvz libftd2xx-x86_64-1.4.8.gz
cd release
cd build
sudo -s 
cp libftd2xx.* /usr/local/lib
chmod 0755 /usr/local/lib/libftd2xx.so.1.4.8
ln -sf /usr/local/lib/libftd2xx.so.1.4.8 /usr/local/lib/libftd2xx.so
exit
```


## Preps
Other packages needed to get things done in Ubuntu:
```bash
sudo apt install build-essential clang bison flex libreadline-dev \
                 gawk tcl-dev libffi-dev git mercurial graphviz \
                 xdot pkg-config python python3 libftdi-dev \
                 qt5-default python3-dev libboost-all-dev cmake libeigen3-dev
```


## IceStorm toolchain
This is get from [Clifford](http://www.clifford.at/icestorm/), go the that page or follow the commands described next.


### IceStorm tools
Install the [IceStorm Tools](https://github.com/YosysHQ/icestorm) (icepack, icebox, iceprog, icetime, chip databases):

```bash
git clone https://github.com/YosysHQ/icestorm.git icestorm
cd icestorm
make -j$(nproc)
sudo make install
```

### Arachne-PNR
Install [Arachne-PNR](https://github.com/cseed/arachne-pnr) (place & route tool, predecessor to NextPNR):

```bash
git clone https://github.com/cseed/arachne-pnr.git arachne-pnr
cd arachne-pnr
make -j$(nproc)
sudo make install
```

### NextPNR
Install [NextPNR](https://github.com/YosysHQ/nextpnr) (place & route tool, Arachne-PNR replacement):

```bash
git clone https://github.com/YosysHQ/nextpnr nextpnr
cd nextpnr
cmake -DARCH=ice40 -DCMAKE_INSTALL_PREFIX=/usr/local .
make -j$(nproc)
sudo make install
```

### Yosys
Install [Yosys](http://bygone.clairexen.net/yosys/) (Verilog synthesis):

```bash
git clone https://github.com/YosysHQ/yosys.git yosys
cd yosys
make -j$(nproc)
sudo make install
```

## Verify whether the IceStorm toolchain works
To double check everything is fine so far, let's test the setup for the kit iCESugar-nano (featuring the  iCE40LP1K-CM36 chip).

Connect the hardware board to the USB and try the `blink demo` from [wuxx](https://github.com/wuxx/icesugar-nano).

The folder is the `https://github.com/wuxx/icesugar-nano/tree/main/src/basic/blink`.


## RISC-V toolchain
Install the [Clifford Wolf's Picorv32](https://github.com/cliffordwolf/picorv32).

```bash
git clone https://github.com/cliffordwolf/picorv32 picorv32

sudo apt install autoconf automake autotools-dev curl libmpc-dev \
         libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo \
         gperf libtool patchutils bc zlib1g-dev git libexpat1-dev

sudo mkdir /opt/riscv32i
sudo chown $USER /opt/riscv32i

git clone https://github.com/riscv/riscv-gnu-toolchain riscv-gnu-toolchain-rv32i
cd riscv-gnu-toolchain-rv32i
git checkout 411d134
git submodule update --init --recursive

mkdir build
cd build
../configure --with-arch=rv32i --prefix=/opt/riscv32i
make -j$(nproc)
