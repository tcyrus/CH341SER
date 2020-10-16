# CH341SER driver

1. [About driver](#about-driver)  
2. [Changes](#changes)  
3. [Tests](#tests)  
4. [Installation](#installation)  
5. [Official website](#official-website)  
6. [Tutorial on Arch Linux](#tutorial-on-arch-linux)  
7. [Compatibility](#compatibility)  

## About driver

It's a manufacturer software of standard serial to usb chip marked CH340

## Changes

Added line  

```c
#include <linux/sched/signal.h>
```

which helps to fix the problem below:  

```
error: implicit declaration of function ‘signal_pending’; did you mean ‘timer_pending’? [-Werror=implicit-function-declaration]
```

and changed line:

```c
wait_queue_t wait;
```

to

```c
wait_queue_entry_t wait;
```

which helps to fix next problem below:

```
error: unknown type name ‘wait_queue_t’; did you mean ‘wait_event’?
```

Additionally first pull request helped to merge changes for version 1.5 released in 2018-03-18:

[juliagoda/CH341SER#1](https://github.com/juliagoda/CH341SER/pull/1)


## Tests

Tested on:
* Arch Linux 4.11.3-1-hardened
* Arch Linux 4.11.3-1-ARCH
* Linux Mint 19.3 Cinnamon
* Ubuntu 18.04.5 LTS

## Installation

See original `readme.txt`

## Official website

[http://www.wch.cn/download/CH341SER_LINUX_ZIP.html](http://www.wch.cn/download/CH341SER_LINUX_ZIP.html)

## Tutorial on Arch Linux

Tested for Arduino UNO R3 Clone

install required packages:

```
sudo pacman -S arduino arduino-docs avr-binutils avr-gcc avr-libc avrdude
```

if your system detects the package below:

```sh
pacman -Qs arduino-avr-core
```

you should remove it:

```sh
sudo pacman -R arduino-avr-core
```

we add current user to uucp and lock groups:  

```sh
gpasswd -a $USER uucp
gpasswd -a $USER lock
```

it's possible, that you have to load that module:  

```
modprobe cdc_acm
```


clone fixed driver:  

```
git clone https://github.com/juliagoda/CH341SER.git
```

according to the original readme.txt, we use the commands below:  

```
make
sudo make load
```

to be sure, that module will be loaded after reboot, you can change file extension from `*.ko` to `*.ko.gz` and add it to drivers path:  

```
find . -name *.ko | xargs gzip
sudo cp ch34x.ko.gz /usr/lib/modules/$(uname -r)/kernel/drivers/usb/serial
```

if the command:

```
lsmod | grep ch341
```

shows some result, then:

```
sudo rmmod ch341
sudo mv /usr/lib/modules/$(uname -r)/kernel/drivers/usb/serial/ch341.ko.gz /lib/modules/$(uname -r)/kernel/drivers/usb/serial/ch341.ko.gz~
sudo depmod -a
```

let's connect Arduino UNO R3 Clone to USB input and check our results:  

```
dmesg | grep ch34x
```


that's example of my command's output:

```
[  492.836159] ch34x 3-1:1.0: ch34x converter detected
[  492.846265] usb 3-1: ch34x converter now attached to ttyUSB0
```


so our driver ch34x was successfully loaded and our port's name is ttyUSB0  


Let's start our installed Arduino IDE.  


First we should install package for Arduino AVR Boards from Boards Manager:

![Choose of Boards Manager](images/arduino_change1.png)

![Installation of packages for Arduino AVR Boards from Boards Manager](images/arduino_change2.png)

And now we must choose our port's name. My port's name is `ttyUSB0`.  

![Choose of port's name](images/arduino_change3.png)


#### Why we wanted arduino-avr-core package from official repo to be removed? Let's check it out.  

```
sudo pacman -S arduino-avr-core
```

and let's reopen our Arduino IDE and check out boards list.  

![New label have been added to boards list](images/arduino_change4.png)

New label "Arch Linux Arduino AVR Boards" with boards has been added to boards list. It has the same options, so let's choose "Arduino/Genuino Uno" from a new part and upload our code to Arduino UNO R3 Clone.

![The same option has been checked in new part](images/arduino_change5.png)

![Upload with error](images/arduino_change6.png)

It didn't work. I think, that the package was created for original arduino boards, which are not compatible with their clones. If you want to have installed the package and work on clone of Arduino, better choose the same option from part labeled "Arduino AVR Boards".


## Compatibility

This driver is not compatible with the Olimex ESP32-POE rev C board
