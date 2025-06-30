# sxb

This repository includes two tools.  The first, `sxb` is for interacting with the W65C02SXB
board by Western Design Center (WDC).  The second, `sxb6` is for the WDC W65C816SXB board
with WDCMON v2.0.

Note the W65C02SXB likely WDCMON v2.0.4.3 which has a different api than the WDCMON v2.0 on the W65C816SXB board.

## Identifying WDCMON

The ROM in my W65C02SXB board starts with this information:
```
00000000  57 44 43 ff 4c 15 81 ff  57 44 43 36 35 63 30 32  |WDC.L...WDC65c02|
00000010  53 4b 20 57 44 43 4d 4f  4e 20 56 65 72 73 69 6f  |SK WDCMON Versio|
00000020  6e 20 3d 20 20 32 2e 30  2e 34 2e 33 56 65 72 73  |n =  2.0.4.3Vers|
00000030  69 6f 6e 20 44 61 74 65  20 3d 20 54 75 65 20 4a  |ion Date = Tue J|
00000040  75 6c 20 20 32 20 32 30  31 33 20 31 36 3a 32 34  |ul  2 2013 16:24|
```

Note the ROM says `WDC65C02SK`, not `W65C02SXB` as might be expected.  Regardless this
version works with the original `sxb` tool by kalj and others.

The ROM in my W65C816SXB doesn't have the same block of text, but does have `SXB6`
embedded near the beginning. The SXB6 value and some version information is returned
by the board info call.

The start of the ROM file looks like below.  I change the code bytes to 'xx' as we are
only interested in the 12 bytes returned by the board info call. The first 4 bytes
are 'SXB6' to identify the board, followed by a 32 bit little endian hardware version
and then a software version.  In this case my hardware is revision 300 and software
is 200.  This should correspond to v3.0.0 hardware (Rev C on the silkscreen on mine) and
WDCMON v2.0. 
```
00000000  xx xx xx xx xx xx xx xx  xx xx xx xx 53 58 42 36  |............SXB6|
00000010  2c 01 00 00 c8 00 00 00  xx xx xx xx xx xx xx xx  |,...............|
```

Of course getting the ROM information above generally means you already ran the `sxb`
or `sxb6` tool, but I put it here for confirmation / clarification.

Since the versions of WDCMON are quite different, I just copied the original `sxb` tool
to `sxb6` (the board identifier in the ROM) and made the modifications to support it.

These two scripts should probably be merged with a flag to select the board type.
That is a future enhancement I'll look at once I'm confident in SXB6 support.

# sxb6 usage

You'll need `python3` with the `pyserial` package installed. 

You can show supported commands with the `-h` option. Currently only `info`, `read`, `write`, and `exec` are supported.  None of the flash commands are implemented.
```
$ ./sxb6 -h
usage: sxb6 [-h] [-d DEVICE] command ...

positional arguments:
  command
    info                Get board info
    read                Read bytes
    write               Write bytes
    exec                Start execution

options:
  -h, --help            show this help message and exit
  -d DEVICE, --device DEVICE
                        Serial device
```

You'll need to pass the device to use, or you could edit the script and change the default line. (Look for CHANGEME)

Under Linux for example you could run `sxb6 -d /dev/ttyUSB0 info` to get the board info.

## info

The `info` command prints the board model.  This should be `SXB6` for the W65C816SXB board.  If/when the W65C02SXB (with WDCMON v2.0) is supported it might be different.

Here you can see my board is `SXB6`, the board revision shows as `300` (my board is a REV.C) which I think would be 3.0.0.   The software / monitor version of `200` should be equivalent to `2.0.0` here.
```
$ ./sxb -d /dev/tty.usbserial-3 info
     Board model: SXB6
  Board revision: 300
Monitor revision: 200
00  53  S
01  58  X
02  42  B
03  36  6
04  2c  ,
05  01
06  00
07  00
08  c8  È
09  00
0a  00
0b  00
```

## read

The `read` command returns `size` bytes starting with address `addr` and dumps them raw to stdout.
If you pass the `--table` option it will dump in hex bytes similar to the `hexdump` command.

```
$ ./sxb6 read -h
usage: sxb6 read [-h] [-t] addr size

positional arguments:
  addr
  size

options:
  -h, --help   show this help message and exit
  -t, --table
```

The command below will dump the WDCMON ROM image itself.

```
$ ./sxb6 -d /dev/tty.usbserial-3 read 0xf800 2048 > wdcmon.bin
```

Using the `--table` argument generates a nice table as shown here.

```
$ ./sxb6 -d /dev/tty.usbserial-3 read --table 0xf800 64
 addr |  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15
 ------------------------------------------------------
 f800 | 4c e6 f9 4c f4 f9 4c 18 fa 4c 34 fa 53 58 42 36
 f810 | 2c 01 00 00 c8 00 00 00 18 fb 78 c2 10 e2 20 a2
 f820 | ff 07 9a a9 00 eb a9 04 85 10 a9 00 9c a1 7f a9
 f830 | ff 8d a0 7f a9 04 8d a1 7f 9c a0 7f 38 6e a0 7f
```

## write
With the `write` command you can load a file into SRAM starting at a specific address.

```
$ ./sxb6 -d /dev/tty.usbserial-3 write 0x2000 demo.bin
```

## exec

The `exec` command executes code by pushing the address to the stack and using `rti` to call the address provided.  The WDCMON command loop is setup as the return address so if you use `rts`
from your code it should return to the monitor.

```
$ ./sxb6 -d /dev/tty.usbserial-3 exec 0x2000
```

# Credits

Forked from the excellent work here: https://github.com/kalj/sxb

With some patches posted to an issue: https://github.com/kalj/sxb/issues/1 

Original credits listed by kalj:
Inspired by https://github.com/andrew-jacobs/dev65 / https://github.com/andrew-jacobs/w65c02sxb-monitor and the python script by forum.6502.org user cjb (see http://forum.6502.org/viewtopic.php?f=4&t=5698).
