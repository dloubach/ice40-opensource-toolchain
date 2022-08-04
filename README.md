# iCE40 Open Source Toolchain
This is a compilation of various sources to create a "how to" build a toolchain environment based on open source using **Linux/Ubuntu 20.04 LTS** distro.

This step-by-step guideline aims to build an open source toolchain for iCE40 series FPGA including:
- IceStorm Tools
   - Arachne-PNR
   - NextPNR
   - Yosys
- icesprog
- RISC-V toolchain 

This is fully based on [Clifford's Project IceStorm](http://www.clifford.at/icestorm/), and inspired on [James Devine's Blog](https://pingu98.wordpress.com/2019/04/08/how-to-build-your-own-cpu-from-scratch-inside-an-fpga/).


## FTDI drivers
Get the FTDI driver from [FTDI chip website](https://ftdichip.com/drivers/d2xx-drivers/), or just follow the next command line block:

```bash
mkdir ftdi
cd ftdi
wget https://www.ftdichip.com/Drivers/D2XX/Linux/libftd2xx-x86_64-1.4.8.gz
tar xfvz libftd2xx-x86_64-1.4.8.gz
cd release/build/
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
git checkout 83b8ef9
make -j$(nproc)
sudo make install
```

### Arachne-PNR
Install [Arachne-PNR](https://github.com/cseed/arachne-pnr) (place & route tool, predecessor to NextPNR):

```bash
git clone https://github.com/cseed/arachne-pnr.git arachne-pnr
cd arachne-pnr
git checkout c40fb22
make -j$(nproc)
sudo make install
```

### NextPNR
Install [NextPNR](https://github.com/YosysHQ/nextpnr) (place & route tool, Arachne-PNR replacement):

```bash
git clone https://github.com/YosysHQ/nextpnr nextpnr
cd nextpnr
git checkout bd137a8b
cmake -DARCH=ice40 -DCMAKE_INSTALL_PREFIX=/usr/local .
make -j$(nproc)
sudo make install
```
The GUI is not built by default in nextpnr. Add `-DBUILD_GUI=ON` to the `cmake` command line, and make sure `qt5-default` is installed.


### Yosys
Install [Yosys](http://bygone.clairexen.net/yosys/) (Verilog synthesis):

```bash
git clone https://github.com/YosysHQ/yosys.git yosys
cd yosys
git checkout 62739f7b
make -j$(nproc)
sudo make install
```

## Verify whether the IceStorm toolchain works
To double check everything is fine so far, let's test the setup.  
Here, I used the hardware kit **iCESugar-nano** v1.2 (featuring the *iCE40LP1K-CM36* chip).

Connect the hardware board to the computer's USB and try the `blink demo` from [wuxx](https://github.com/wuxx/icesugar-nano).

The folder is the `https://github.com/wuxx/icesugar-nano/tree/main/src/basic/blink`.

```bash
git clone https://github.com/wuxx/icesugar-nano
cd icesugar-nano/src/basic/blink
git checkout c45b20b
make build
make prog_flash
```
Hopefully, the orange LED in the iCESugar-nano should blink.  
The `prog_flash` option copies the `.bin` file to the `iCELink` folder in the the USB-based mass storage device created once the iCESugar-nano is pluged in the computer.


# icesprog
This program is used to flash or do more configs (see the help `icesprog -h`).  
Make sure your iCESugar-nano is connected to the USB.  
Do the following:

```bash
sudo apt install libhidapi-dev
sudo apt install libusb-1.0-0-dev
git clone https://github.com/wuxx/icesugar.git icesugar
cd icesugar/tools/src/
git checkout c45b20b
make
sudo cp icesprog /usr/local/bin/.
icesprog -p 
```

with the last command, you should see:
```bash
probe chip
board: [iCESugar-Nano]
flash: [w25q16] (2MB)
done
```


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
```

## Femto SoC RV32 (*work in progress*)
Follow the next command line to have your *Femto SoC RV32* synthesis, PnR, and bitstream `femtosoc.bin` for **iCESugar-nano v1.2**. This step is originally from [Bruno Levy](https://github.com/BrunoLevy/learn-fpga), but we use the modifications as in [here](https://github.com/dloubach/femtorv32.git).

```bash
sudo apt install python3-serial
git clone https://github.com/dloubach/femtorv32.git femtorv32
cd femtorv32/FemtoRV/
make icesugar_nano
```
iCESugar-nano hw board | nextpnr running
---------------------- | ---------------
Featuring FPGA iCE40LP1K-CM36 | Logic Cells (LUT + FF) utilization: 1,248/1,280 (97%)
![icesugar-nano-hw](https://github.com/dloubach/ice40-opensource-toolchain/blob/master/icesugar-nano.jpeg) | ![nextpnr](https://github.com/dloubach/ice40-opensource-toolchain/blob/master/ice40lp1k-cm36.gif)



