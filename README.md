# ARM Control Framework
This framework provides external control of an embedded device through a UART connection. Two key components to this framework are the: request handler and commands class.

The request handler utilizes a UART connection to send and received data in the form of packets. A complete packet of data should consist of one command and 0-to-n number of parameters. The command field determines the function to invoke.

The commands class is where callable functions live. This class is independent of the request handler, except for a callback pointer. This pointer enables UART responses from class functions. Since the class is independent, adding new functionality straight forward. 

<img src="https://github.com/jongreene/projects-media/blob/master/acorns/6db4ab84-d0be-4a54-b1a0-f3dc903e4d98.jpeg?raw=true" width="300" title="Rover on foam pad"></img>

Setup
---
Following this guide should be fairly straight forward as long as you have the following tools installed:
- [CMake](https://cmake.org/download/) 
- [J-Link Software](https://www.segger.com/downloads/jlink/#J-LinkSoftwareAndDocumentationPack)
##### Cloning
Open a terminal and navigate to where you want the repository to reside. From the terminal:
```bash 
  # clone the repository
  $ git clone https://github.com/jongreene/semi-autonomous-rover.git
```

##### Compiling
Before we can start compiling, we need to prepare the project. From the same terminal:
```bash
  # create a RELEASE directory
  $ mkdir RELEASE

  # enter that directory
  $ cd RELEASE
```
We're now ready to start generating the build files. 
> The first time running cmake in the repository will take a long time, especially if you have a slow internet connection. This is due to the cmake build system downloading ARM GNU toolchain and cloning Mbed OS into the project. Cmake then applies the appropriate patches to the downloaded resources. 
From the same terminal (in the REEASE directory):
```bash
  # resolve dependencies and generate build files
  $ cmake ..

  # compile STK3700 binary
  $ make
```

##### Flashing
> I will be using a standalone J-Link JTAG/SWD programmer to flash binaries onto the board (this requires the mcu to be unlocked using Silabs software). You are free to flash with the onboard J-Link programmer but it will not be covered.

First attach you programmer to the development board and supply the board power. Shown below.

<img src="https://github.com/jongreene/projects-media/blob/master/mbed-framework/board_dev_setup.jpg?raw=true" title="Rover on foam pad"></img>

From a terminal pointed at the build directory, enter the commands below.
```bash
  # connect to board
  $ JLinkExe -device EFM32GG380F1024 -if SWD -speed 4000

  # load the binary onto the STK3700
  $ loadbin bin/semi-autonomous-rover.bin, 0x0

  # reset and start the board
  $ r
  $ g
```

The board should now be running the new binary.

Communication
---
The rover is setup to work seamlessly with [this](https://www.adafruit.com/product/2479) bluetooth module but will work with any standard UART device.

##### Setup
Below you will find images that will guide you through setting up a UART connection between the Giant Gecko and a PC running CuteCom.

<img src="https://github.com/jongreene/projects-media/blob/master/mbed-framework/UART_wiring.jpg?raw=true" title="Rover on foam pad"></img>

> Connection of the UART-to-USB adapter.

<img src="https://github.com/jongreene/projects-media/blob/master/mbed-framework/cutecom_settings.jpg?raw=true" title="Rover on foam pad"></img>

> CuteCom using the rover's default baud rate of 9600bps.

<img src="https://github.com/jongreene/projects-media/blob/master/mbed-framework/cutecom_example.jpg?raw=true" title="Rover on foam pad"></img>

> Example usage of commands

It is important the each line is sent with a new line character `\n`. This character signals to the Giant Gecko's UART parser then end of a command.

Commands
---
Commands follow a simple single-bracket matching format, i.e. {...}, to verify a full command was received.
##### Command format
```bash
  input: {command_name,param1,param2,...}\n 
```
> The newline character `\n` is displayed for illustration only and should be generated by your UART client.

##### Command responses
There are two types of responses that can be expected when issuing a command. 
- An acknowledgement that a line (`\n`) of data was received, and whether or not that data was properly formatted and requesting a valid function. 
- A response generated by the function being invoked and can happen as many times as desired, so long as the function is still executing.

##### Command error responses
```
Badly formatted json: 
  {"nak":"","payload":{"return_value":false,"return_string":"json error: badly formatted json"}}
Invalid command: 
  {"notification":"payload":{"return_string":"error: command not found"}}
Command queue overflow: 
  {"notification":"payload":{"return_string":"error: queue overflowed"}}
```
##### Example command
```bash
  input: {drive,40,40,20,20}
  output: {""}
  output: {""}
```

Customization
---
This project is open source, so customization is absoultely possible and encouraged. The easiest place to start is by creating some custom commands and calling them over UART. You'll find that it's surprisingly easy to add new functionality, once you have your development environment configured.
##### Creating commands
> Through the use of a highly modularized C/C++ framework, defining new callable functions takes little to no understanding of the underlying services. New routines can be implemented in just a few steps, making driver development fast and simple. While this rover is very capable, the underlying framework that drives it is the true bread and butter of this project.
