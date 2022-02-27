# Rpi

This repository is designated for Embedded Linux Kernel Modules written for Raspberry Pi and its various peripherals

GPIO

PREREQUISITES
To follow this, you need:

Raspberry Pi 3 board full;
Micro USB cable;
8 GB Micro SD card;
USB-to-Serial debug module for Raspberry Pi 3 or USB to TTL adapter;
A PC provided with Ubuntu Desktop 14.04 LTS, or a virtual machine hosting Ubuntu Desktop 14.04 LTS;
A Micro SD card reader attached to the PC/virtual machine;
(Optional) Micro HDMI cable.
INTRODUCTION
The Raspberry Pi 3 offers a number of I/O, which are accessible through the connectors labeled J8 


In this Lab, a loadable kernel module is implemented that programs GPIO21 as output and GPIO20 as input. Moreover, every time the value on the GPIO20 input changes, an interrupt is generated and handled to read the updated value.

In this Lab, the GPIO-based I/O is used as described in the lecture in module 6 and, in particular, GPIO21 and 20 will be used. The two-integer values identify univocally within Linux two hardware resources of the BCM2837 processor that powers the Raspberry Pi 3 board. The correspondence between hardware resource and Linux identifier is obtained using the following equation:

Linux ID =  Bit Number

You will need to setup a yocto build catered to the Raspberry Pi 3 and prep a build environment, similar to the following below

` cd ~/raspberryPi3 `
` source sources/poky/oe-init-build-env rpi-build `

Similar to the files created during the previous lab, create and move to the directory raspberrypi3/sources/meta-raspberrypi/recipes-kernel/gpio-mod/files and make a file named gpio.c with the content seen in this repo

Once these operations are completed, you have to tell the machine layer configuration that the new driver is needed. For this purpose, edit the file

` cd ~/raspberrypi3/rpi-build/conf/local.conf `

Add the following statement

` IMAGE_INSTALL:append += "gpio" `

You are now ready to build the new system as follows:

cd ~/raspberrypi3/rpi-build
bitbake -c clean rpi-basic-image
bitbake rpi-basic-image

During the build, the compiler will recognize a discrepancy in the license reference. Copy and paste the suggested reference (“the new md5 checksum is..”) into your gpio_1.0.bb file and start the build again.


RUNNING THE MODULE
After booting the new Linux system, you can check whether the build process was completed successfully. After logging into the Raspberry Pi 3, you can type the following commands:

root@raspberrypi3:/# ls -la /lib/modules/4.1.21/kernel/drivers/my_mod
drwxr-xr-x    2 root     root          1024 May 15 08:47 .                     
drwxr-xr-x   43 root     root          1024 May 10 12:09 ..                    
-rw-r--r--    1 root     root          9080 May 12 03:02 gpio.ko        
-rw-r--r--    1 root     root          5384 May 11 08:05 hello.ko 
root@raspberrypi3:/#

The directory in the root file system contains gpio.ko, which is the kernel object containing the binary code for the loadable kernel module.

You can now insert the module in the kernel as follows:

root@raspberrypi3:/# insmod /lib/modules/4.1.21/kernel/drivers/my_mod/gpio.ko
[   1981.635413] Loading gpio_module
[   1981.638775] 243:0

The output messages indicate that the module has been loaded correctly, with major number 243 (This number may vary on your system - take it into account in the following steps) and minor number 0.

To test the module, you have to first create the associated device file:

root@raspberrypi3:/# mknod /dev/gpio c 243 0

You can now communicate with the module through the Linux command line.

As an example, you can issue the following command, observing the following output:

root@raspberrypi3:/# echo 1 > /dev/gpio
[ 2150.247267] gpio_dev: gpio_open
[ 2150.250630] gpio_dev: InterruptHandler got GPIO_IN with value 1
[ 2150.256641] gpio_dev: gpio_write wrote 1 to GPIO_OUT
[ 2150.261938] gpio_dev: gpio_close

The messages displayed by the module indicate that the echo user application opens the connection with the module through the device file, then it sends the data by invoking the write function of the virtual file system, and finally, it closes the connection. If you wired output GPIO21 to input GPIO20, you will see that the interrupt is triggered and the InterruptHandler function is executed, reading the written value.

To test the read operation, we can write a simple test application called test_gpio.c (also in this repo)


The code shall be built on the host machine, issuing the following command:

arm-linux-gnueabihf-gcc -o test_gpio test_gpio.c

The obtained program, test_gpio, shall be executed on the Raspberry Pi 3 target, by copying it from the development host onto a USB stick that will then be plugged into the Raspberry Pi 3 USB port.

When connecting the USB to the Raspberry Pi, mount it to the media folder with the mount command:

mount /dev/sda1 media

If your USB is not called sda1, use the following command to find its name:

fdisk -l

The test program can be executed as follows (ensure /dev/cgpio exists as done earlier):

root@raspberrypi3:~# ./media/test_gpio > tmp
[  947.552708] gpio_dev: gpio_open
[  947.556149] gpio read (count=1, offset=0)
[  947.562213] gpio_dev: gpio_close

The test application interacts with the driver through the virtual file system, invoking the open, read, and release functions.

The content of the tmp file contains the read value from the module. As an example

root@raspberrypi3:~# echo 1 > /dev/gpio
[ 1068.239500] gpio_dev: gpio_open
[ 1068.243109] gpio_dev: InterruptHandler got GPIO_IN with value 1
[ 1068.250988] gpio_dev: gpio_write wrote 1 to GPIO_OUT
[ 1068.256276] gpio_dev: gpio_close

root@raspberrypi3:~# ./media/test_gpio > tmp
[ 1071.322513] gpio_dev: gpio_open
[ 1071.325956] gpio read (count=1, offset=0)
[ 1071.332148] gpio_dev: gpio_close
root@raspberrypi3:~# cat tmp
read: 1

We can see that the content of tmp corresponds to the value (1) sent to the module.

For removing the module, you can act as follows:

root@raspberrypi3:/# rmmod gpio
[  240.457807] Cleaning-up gpio_dev.

The message indicates that the module cleanup function has been executed correctly.