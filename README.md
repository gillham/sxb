# sxb
Tool for interacting with W65C816SXB board with WDCMON v2.0

# usage

You'll need `python3` with the `pyserial` package installed. 

You can show supported commands with the `-h` option. Currently only `info`, `read`, `write`, and `exec` are supported.  None of the flash commands are implemented.
```
$ ./sxb -h
usage: sxb [-h] [-d DEVICE] command ...

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

Under Linux for example you could run `sxb -d /dev/ttyUSB0 info` as an example.

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
$ ./sxb read -h
usage: sxb read [-h] [-t] addr size

positional arguments:
  addr
  size

options:
  -h, --help   show this help message and exit
  -t, --table
```

The command below will dump the WDCMON ROM image itself.

```
$ ./sxb -d /dev/tty.usbserial-3 read 0xf800 2048 > wdcmon.bin
```

Using the `--table` argument generates a nice table as shown here.

```
$ ./sxb -d /dev/tty.usbserial-3 read --table 0xf800 64
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
$ ./sxb -d /dev/tty.usbserial-3 write 0x2000 demo.bin
```

## exec

The `exec` command executes code by pushing the address to the stack and using `rti` to call the address provided.  The WDCMON command loop is setup as the return address so if you use `rts`
from your code it should return to the monitor.

```
$ ./sxb -d /dev/tty.usbserial-3 exec 0x2000
```

# Credits

Forked from the excellent work here: https://github.com/kalj/sxb

With some patches posted to an issue: https://github.com/kalj/sxb/issues/1 

Original credits listed by kalj:
Inspired by https://github.com/andrew-jacobs/dev65 / https://github.com/andrew-jacobs/w65c02sxb-monitor and the python script by forum.6502.org user cjb (see http://forum.6502.org/viewtopic.php?f=4&t=5698).
