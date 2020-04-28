Vitis Project files
===================

### Important patch for Vitis 2019.2

To use the build script in this directory, you must first apply the following patch
to your Vitis installation:

https://www.xilinx.com/support/answers/73252.html

### How to build the Vitis workspace

In order to make use of these source files, you must first generate
the Vivado project hardware design (the bitstream) and export the hardware.
Check the `Vivado` folder for instructions on doing this from Vivado.

Once the bitstream is generated and exported, then you can build the
Vitis workspace using the provided `build-vitis.tcl` script.

### Scripted build

The Vitis directory contains a `build-vitis.tcl` script which can be run to automatically
generate the Vitis workspace. Windows users can run the `build-vitis.bat` file which
launches the Tcl script. Linux users must use the following commands to run the build
script:
```
cd <path-to-repo>/Vitis
/<path-to-xilinx-tools>/Vitis/2019.2/bin/xsct build-vitis.tcl
```

The build script does four things:

1. Makes a copy of the `axipcie` driver from 
`{Vitis Install Dir}\data\embeddedsw\XilinxProcessorIPLib\drivers\` to the repo's local 
directory `\EmbeddedSw\XilinxProcessorIPLib\drivers\`. Files that are already there
as part of the repo are not overwritten, which allows us to keep a modified version
of the driver. This modified version of the driver is used by the projects using the
Gen3 core (AXI Bridge for PCIe Gen3 IP). See below for more information.
2. Generates a Hello World example application for each exported Vivado design
that is found in the ../Vivado directory. Most users will only have one exported
Vivado design.
3. Deletes the `helloworld.c` source file from the application.
4. Copies the appropriate enumeration application source file from the
`\Vitis\common\src\` directory of this repo into the application source directory.

### Run the application

1. Open Xilinx Vitis.
2. Power up your hardware platform and ensure that the JTAG is
connected properly.
3. Select Xilinx Tools->Program FPGA. You only have to do this
once, each time you power up your hardware platform.
4. Click Run from the toolbar to run your application. You can modify the code
and click Run as many times as you like, without going through
the other steps.

### Linker script modifications for Zynq designs

For the Zynq designs, the Vitis's linker script generator automatically assigns all sections
to the BAR0 memory space, instead of assigning them to the DDR memory space. This causes 
failure of the application to run, when booted from SD card or JTAG. To overcome this problem,
the Vitis build script modifies the generated linker script and correctly assigns the sections
to DDR memory.

If you want to manually create an application in the Vitis for one of the Zynq designs,
you will have to manually modify the automatically generated linker script, and set all sections
to DDR memory.

### Driver for AXI Bridge for PCIe Gen3 IP

Some of the Vivado designs in this project use the AXI Memory Mapped to PCIe Gen2 IP
and others use the AXI Bridge for PCIe Gen3 IP. Vitis comes with a driver for the Gen2
core that is called `axipcie`. The BSPs for projects using the Gen2 core refer to that 
driver. You can find the driver sources in the Vitis installation files:

`{Vitis Install Dir}\data\embeddedsw\XilinxProcessorIPLib\drivers\`

The Vitis does not currently supply a driver for the Gen3 core, so we have to create our
own. Luckily, there are enough similarities between the Gen2 and Gen3 cores that we can 
get away with using a modified version of the `axipcie` driver on the Gen3 core. This 
will allow us to do some simple things such as link-up detection, determining link speed
and width, and enumerating PCIe devices.

We create this "Gen3 version" of the driver by making a local copy of the `axipcie` driver
sources and modifying the `.mdd` file, specifying that the driver supports the Gen3 core.
For Vitis to be aware of our locally copied driver, we set the Vitis's repository path to the path 
of the driver. The `build-sdk.tcl` script handles the copying and modification of the 
`axipcie` driver, which is stored locally in the `EmbeddedSw/XilinxProcessorIPLib/drivers` 
directory.

### UART configurations

* Zynq & ZynqMP designs: 115200 baud
* Microblaze designs: 9600 baud

