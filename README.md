# Reason for this fork.
Using Raspberry pi 4 model B. \
Image created with `Raspberry Pi Imager v1.7.3`.
Image used : `Raspberry Pi OS (32-bit)`

    Raspbian GNU/Linux 11 (bullseye)"
    debian_version 11.6
    Linux raspberrypi 5.15.76-v7l+


XVCpi from [derekmulcahy](https://github.com/derekmulcahy/xvcpi) was compiling fine on the pi, but bitfiles were not transfering due to the following errors: 
    
    ERROR: [Labtools 27-3165] End of startup status: LOW 
    ERROR: [Common 17-39] 'program_hw_devices' failed due to earlier errors.

XVCpi from [le-petit-prince](https://github.com/le-petit-prince/xvcpi) looked promising, but it was not compiling. So the makefile was modified by changing order of the arguments.

# Xilinx Virtual Cable Server for Raspberry Pi

[Xilinx Virtual Cable](https://github.com/Xilinx/XilinxVirtualCable/) (XVC) is a TCP/IP-based protocol that  acts like a JTAG cable and provides a means to access and debug your  FPGA or SoC design without using a physical cable.
A full description of Xilinx Virtual Cable in action is provided in the [XAPP1252 application note](https://www.xilinx.com/support/documentation/application_notes/xapp1251-xvc-zynq-petalinux.pdf).

**Xvcpi** implements an XVC server to allow a Xilinx FPGA or SOC to be controlled remotely by Xilinx Vivado using the Xilinx Virtual Cable protocol. Xvcpi uses TCP port 2542.

The **xvcpi** server runs on a Raspberry Pi which is connected, using JTAG, to the target device. **Xvcpi** bitbangs the JTAG control signals on the Pi pins. The bitbanging code was originally extracted from [OpenOCD](http://openocd.org).

# Wiring
Note: The Raspberry Pi is a 3.3V device. Ensure that the target device and the Pi are electrically compatible before connecting. 100 Ohm resistors may be placed inline on all of the JTAG signals to provide a degree of electrical isolation.

JTAG uses 4 signals, TMS, TDI, TDO and, TCK.
From the Raspberry Pi perspective, TMS, TDI and TCK are outputs, and TDO is an input.
The default pin mappings for the Raspberry Pi header are:
```
TMS=25, TDI=10, TCK=11, TDO=9
```
The pin mappings can be changed by optional flags of **xvcpi**: -c(for TCK), -m(for TMS), -i(for TDI), -o(for TDO). 
```
./xvcpi -c 22 -m 23 -i 24 -o 25
```
Above command changes the pin mappings to:
```
TMS=23, TDI=24, TCK=22, TDO=25
```

In addition a ground connection is required. Pin 6 of the Model B+ 40-Pin header is a conveniently placed GND.

Note that XVC does not provide control of either SRST or TRST and **xvcpi** does not support a RST signal.

# Usage
Start **xvcpi** on the Raspberry Pi. An optional -v flag can be used for verbose output.

Vivado connects to **xvcpi** via an intermediate software server called hw_server. To allow Vivado "autodiscovery" of **xvcpi** via hw_server run:

```
hw_server -e 'set auto-open-servers xilinx-xvc:<xvcpi-server>:2542'
```

Alternatively, the following tcl commands can be used in the Vivado Tcl console to initiate a connection.

```
connect_hw_server
open_hw_target -xvc_url <xvcpi-server>:2542
```

Full instructions can be found in [ProdDoc_XVC_2014 3](ProdDoc_XVC_2014_3.pdf).

# Snickerdoodle
The initial purpose of **xvcpi** was to provide a simple means of programming the [Snickerdoodle](http://snickerdoodle.io).
# Licensing
This work, "xvcpi.c", is a derivative of "xvcServer.c" (https://github.com/Xilinx/XilinxVirtualCable)

"xvcServer.c" is licensed under CC0 1.0 Universal (http://creativecommons.org/publicdomain/zero/1.0/)
by Avnet and is used by Xilinx for XAPP1251.

"xvcServer.c", is a derivative of "xvcd.c" (https://github.com/tmbinc/xvcd)
by tmbinc, used under CC0 1.0 Universal (http://creativecommons.org/publicdomain/zero/1.0/).

Portions of "xvcpi.c" are derived from OpenOCD (http://openocd.org)

"xvcpi.c" is licensed under CC0 1.0 Universal (http://creativecommons.org/publicdomain/zero/1.0/)
by Derek Mulcahy.