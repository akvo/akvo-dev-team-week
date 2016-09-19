# Getting started with NodeMCU and mqtt

## About NodeMCU
NodeMCU is a microcontroller based on the ESP8266 chip providing wifi connectivity. You can add a firmware with a Lua interpreter (or MicroPython, Lisp, Arduino, JavaScript for example; pick your poison).

## 1. USB drivers

You will need a USB driver, either for _CH340G_ or for _CP2102_  (depending on what type of NodeMCU you are using) to be able to communicate with your NodeMCU. (CH340G and CP2101 are USB to serial converter chips.)

The pcb that you got at DevWeek 2016 uses the _CH340G_ chip.

### Windows

* [CH340G Windows drivers](https://codebender.cc/static/walkthrough/page/3)

* ([Silabs USBtoUART Bridge](http://www.silabs.com/products/mcu/Pages/USBtoUARTBridgeVCPDrivers.aspx?#windows))


### MacOSx

* [CH340G Mac drivers](http://blog.codebender.cc/2015/06/12/new-stuff-updated-mac-drivers/)

* ([Silabs USBtoUART Bridge](http://www.silabs.com/products/mcu/Pages/USBtoUARTBridgeVCPDrivers.aspx?#osx))


## 2. New Firmware
These little buggers come preinstalled with a firmware that takes AT commands, so they're pretty useless until you've fixed them by installing the firmware that lets you write Lua code on them. (There are other options available too, for example using an Arduino firmware.)

### Firmware build service here
This is the [NodeMCU custom builds service](https://nodemcu-build.com/index.php) that lets you select what firmware modules you want and then build a new firmware that you can later download and burn to your NodeMCU.


#### These modules are good to have
* DHT
* mqtt

### Windows
Use [NodeMCU-flasher](https://github.com/nodemcu/nodemcu-flasher)

Install esp_init_data_default.bin at 0x1FC000 and your new firmware at 0x00000. Set SPI mode to QIO.

You can also use esptool.py. (Command line parameters can be found with -h or by reading Mark's intro further down in this doc. The specific entry point addresses are the same as above though, 0x1FC000 for the bootloader-thingy and 0x00000 for your firmware. SPI mode to QIO.)
```
git clone https://github.com/themadinventor/esptool.git
cd esptool
python esptool.py -h
```

## 3. ESPlorer
Install ESPlorer and scan for the device. Then connect. (From start your device will be a 115 600 baud device with AT-based firmware, but since you've already flashed a new firmware on it, you can now enjoy Lua in all it's glory.)

ESPlorer can be found [here](https://github.com/4refr0nt/ESPlorer)

### Talking to the NodeMCU with ESPlorer
* Connect to the NodeMCU (scan + connect)
* Go to code snippets to start coding.
* You can also issue commands in the terminal to the right in the window.

### Talking to the NodeMCU with a terminal program
* Connect the device with a USB micro cable.
* Open the device manager and check out what COM port number the device got.
* Open a terminal program (the built in or PuTTY for example), set it to 9600 baud rate and the correct COM port and then start chatting away!

### Problems?
Connecting to the device can be a bitt fiddly, but there are some tricks to it that can help you if you get stuck.

* Try to build a smaller firware. Use fewer libraries. (The device is _very_ limited in memory and Flash and sometimes adding to many libs just kills it.)
* Try to reset it a couple of times and try to change the baud rate. It doesn't always autoconfigure well.


### Documentation for Lua for NodeMCU
[Documentation](https://nodemcu.readthedocs.io)

### Example:
```
-- Connect to the local wifi network.
wifi.sta.config("WifiName", "WifiPassword", 1)

```



## 4. Setting up an mqtt server (generic + Linux)
Downloads for Mosquitto can be found [here](http://www.eclipse.org/mosquitto/download/) or install it with apt-get, like this:

```
sudo apt-add-repository ppa:mosquitto-dev/mosquitto-ppa
sudo apt-get update
sudo apt-get install mosquitto
sudo apt-get install mosquitto-clients
```

### Mosquitto for Windows
Download the precompiled mosquitto server [here](http://mosquitto.org/download/)

#### Windows dependencies

* OpenSSL

    * Download [here](http://slproweb.com/products/Win32OpenSSL.html)
    * Install "Win32 OpenSSL <version>"
    * Required DLLs: `libeay32.dll ssleay32.dll`
* pthreads
    * Download [here](ftp://sourceware.org/pub/pthreads-win32)
    * Install "pthreads-w32-<version>-release.zip
    * Required DLLs: pthreadVC2.dll

Please ensure that the required DLLs are on the system path, or are in the same directory as
the mosquitto executable.

### Mosquitto for MacOSx
```
brew install mosquitto
```


## Clients for trying out mqtt

* A JavaFX based client called [mqttfx](http://www.jensd.de/apps/mqttfx/1.1.0/)
* A [Chrome plugin](https://chrome.google.com/webstore/detail/mqttlens/hemojaaeigabkbcookmlgmdigohjobjm). The most simple option.


## Next steps
http://www.foobarflies.io/a-simple-connected-object-with-nodemcu-and-mqtt/




# MacOSx recipe

[10:42:58] Mark Westra: My recipe:

The Firmware for the nodeMCU consists of two parts:
* An init data block esp_init_data_default.bin
* A part that contains the compiled modules (ends in ...float.bin)

It is important to realise that for each new version of the SDK, a new init data block is needed. I have followed http://nodemcu.readthedocs.io/en/latest/en/flash/#upgrading-firmware to do this. It means:

0) install esptool:
```
$ git clone https://github.com/themadinventor/esptool.git
$ cd esptool
$ sudo python ./setup.py install
```

And go to the folder esptool.

0) install USB driver
http://www.silabs.com/products/mcu/Pages/USBtoUARTBridgeVCPDrivers.aspx?#osx

1) Use the cloud build service to build the firmware with the modules you want: https://nodemcu-build.com/ . This downloads the firmware code from https://github.com/nodemcu/nodemcu-firmware/releases, and gives you a UI to select modules. For example, start with selecting the default ones, plus i2c, mqtt, spi, and 1-wire. This gives you a firmware file called nodemcu-master-11-modules-2016-08-22-13-07-35-float, for example. Put it in a folder /firmware

2) Download the nodeMCU SDK patch 1.5.4.1 (http://bbs.espressif.com/download/file.php?id=1572)(which is the latest one) and extract esp_init_data_default.bin from there. Put it in a folder /firmware.

3) run
$ esptool.py --port <serial-port-of-ESP8266> erase_flash
In my case, this is
$ esptool.py --port=/dev/cu.SLAB_USBtoUART erase_flash

4) put the nodeMCU unit in flash mode, by holding down the "FLASH" button on the device and hit the "RST" button once while doing so.

4) run
$ esptool.py --port <serial-port-of-ESP8266> write_flash <flash options> 0x00000 <nodemcu-firmware>.bin <init-data-address> esp_init_data_default.bin
According to the website mentioned above, the init address in my case is 0x3fc000 for a ESP-12E chipset. So in total, this becomes:
```
$ esptool.py --port=/dev/cu.SLAB_USBtoUART write_flash -fm=qio -fs=32m 0x00000 ../firmware/nodemcu-master-11-modules-2016-08-22-13-07-35-float.bin 0x3fc000 ../firmware/esp_init_data_default.bin
```

4.5) You might need to try 0x1FC000 for the device that you got.

5) To communicate with the device, install Esplorer (http://esp8266.ru/esplorer/), and run it.

6) Choose /dev/cu.SLAB_USBtoUART as the port. Choose 115200 as baud rate, and click 'open'

7) go to the 'nodeMCU and micropython' tab, and click 'chipID'. If this prints an ID, everything should be fine. If you click 'Restart ESP', for example, I get this:
NodeMCU custom build by frightanic.com
 branch: master
 commit: 95e85c74e7310a595aa6bd57dbbc69ec3516e9c6
 SSL: false
 modules: file,gpio,i2c,mqtt,net,node,ow,spi,tmr,uart,wifi
 build  built on: 2016-08-22 13:06
 powered by Lua 5.1.4 on SDK 1.5.4.1(39cb9a32)

8) Running Lua scripts:
The ESP8266 with the NodeMCU firmware executes Lua files. When booting the device it will look for a file called init.lua and execute this file. But first we will clear the device of every file.
 * In ESPlorer, click the format button to delete all files
 * in ESplorer, create a blank script, and put this in it (without the === parts):

```
 ======================= start code ==================================
 -- Config
local pin = 4            --> GPIO2
local value = gpio.LOW
local duration = 1000    --> 1 second


-- Function toggles LED state
function toggleLED ()
    if value == gpio.LOW then
        value = gpio.HIGH
    else
        value = gpio.LOW
    end

    gpio.write(pin, value)
end


-- Initialise the pin
gpio.mode(pin, gpio.OUTPUT)
gpio.write(pin, value)

-- Create an interval
tmr.alarm(0, duration, 1, toggleLED)
=================================== end code ===============================
```

* Save the file as “init.lua”. Save to ESP (I used the upload button in the end). Check if the file has arrived by going to the command tab and clicking 'list files'. restart the module by clicking 'Restart ESP'. The module should start blinking now.
* More info at https://nodemcu.readthedocs.io/en/dev/en/upload/

9) follow http://www.cnx-software.com/2015/10/29/getting-started-with-nodemcu-board-powered-by-esp8266-wisoc/ to create a web server, allowing you to turn on and off the lights.
[10:43:15] Mark Westra: More helpful links:
https://github.com/nodemcu/nodemcu-devkit/wiki/Getting-Started-on-OSX
http://frightanic.com/iot/comparison-of-esp8266-nodemcu-development-boards/
https://github.com/themadinventor/esptool
https://odd-one-out.serek.eu/esp8266-nodemcu-getting-started-hello-world/

## MacOS recipe addendum (by Gabriel)

In Riga I couln't get my NodeMCU to work. I followed the above and tried some variations, but only got the fast blinking LED of a broken flash. When I came back home I sat down and tried again. These are the clues that led me to a working flashing.

After having tried different variants of the above and failed I started looking around at the docs to see if I could figure out what the problem might be. I suspected that the different units could have different hardware, in turn requiring defferent flashing parameters.

In the esptool docs there's a [link](https://github.com/themadinventor/esptool#read-spi-flash-id) to a [flashrom header file](http://code.coreboot.org/p/flashrom/source/tree/HEAD/trunk/flashchips.h) with manufacturer name and part number.

Running esptool flash_id on my device yeilded:

```
-> % esptool.py --port /dev/cu.wchusbserial1410 flash_id
esptool.py v1.2-dev
Connecting...
Manufacturer: ef
Device: 4016
```
This led me [here](https://code.coreboot.org/p/flashrom/source/tree/HEAD/trunk/flashchips.h#L888) and ultimately [here](https://code.coreboot.org/p/flashrom/source/tree/HEAD/trunk/flashchips.h#L899). Googling "W25Q32BV" led me [to a PDF from the chip maufacturer](https://www.winbond.com/resource-files/w25q32bv_revi_100413_wo_automotive.pdf) with detailed specs. The key piece of info was that it's a 32Mbit chip. At the bottom of [this page in the NodeMCU docs](http://nodemcu.readthedocs.io/en/latest/en/flash/#upgrading-firmware) the address for esp_init_data_default.bin is listed.

With this info in hand I got the paramteres right:

```
esptool.py --port /dev/cu.wchusbserial1410 write_flash -fm=qio -fs=32m 0x00000 firmware/nodemcu-master-9-modules-2016-09-15-12-38-36-float.bin 0x3fc000 firmware/esp_init_data_default.bin
```

The important bits I changed were ```-fs=32m``` and ```0x3fc000 firmware/esp_init_data_default.bin``
