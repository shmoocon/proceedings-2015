# NSA Playset: USB Tools

## Abstract

USB implants were among the most talked about gadgets in the NSA ANT catalog
after it leaked last year. Concealed in cables and connectors, these devices
appear to be designed primarily to provide covert communication channels to
malware operating on a host computer. Secondarily it seems that they could be
used to implement USB attacks or to monitor connected USB devices.

We demonstrate tools we've built for the same capabilities, including USB
man-in-the-middle with Daisho, our SuperSpeed USB platform for wired
communication security research. Additionally we show capabilities recently
added to USBProxy, a software framework that can operate on existing hardware
platforms such as BeagleBone Black. Finally we introduce TURNIPSCHOOL, our own
RF implant hidden in a USB cable.

## Introduction

USB is the most common interface for connecting devices to host computers, with
more than 2 billions devices sold per year. With such a large number and wide
variety of devices, there comes a vast surface area to examine for security
vulnerabilities.

We present three projects for exploring the USB protocol, devices, and host
interfaces. They include packet sniffers which are able to monitor and modify
traffic in transit for both High Speed and Super Speed USB interfaces and a
covert USB-attached wireless microcontroller.

All three projects feature open source hardware and software, allowing any
interested researcher or hobbyist to experiment freely with the USB protocol.

## USBProxy

USBProxy[^1], first announced at ShmooCon 2014[^2], is userspace tool for monitoring, modifying, and injecting USB packets in a connection. It was designed as a software solution for inexpensive, off-the-shelf hardware, such as the BeagleBone Black, to enable anyone to examine USB communications.

Since the initial demonstration of USBProxy there has been a great deal of interest in using it to reverse engineer USB based protocols, especially for systems where access to the host is not possible. For example both Xbox 360[^3] and Wii U controller[^4] protocols have been examined with USBProxy.

We have also had success with standard USB protocols. For example, our work with USB Mass Storage[^5] has allowed us to inspect and block unwanted writes from suspicious hosts.

Our latest step in the effort of opening USB experimentation to a wider audience has been to introduction of Python bindings to allow devices and packet filters to be written in Python. In turn this has allowed us to support a library of existing device implementations that have historically targeted the FaceDancer platform[^6]. We were able to demonstrate an unmodified FaceDancer keyboard implementation running on USBProxy and the BeagleBone Black.

## Daisho: USB 3.0 Super Speed

Daisho[^7] [^8] is a DARPA Cyber Fast Track (CFT)[^9] funded open source hardware development platform for analyzing and attacking wired interfaces. USB 3.0 Super Speed monitoring and man-in-the-middle attack is a primary goal of the Daisho project.

The Daisho circuit board ("main board") contains an FPGA and a USB 3.0 Super Speed physical layer interface (PHY) for monitoring and control. For USB 3.0 work, a "front end" circuit board containing two additional PHYs is attached to the main board. These additional ports attach to a host and target device. The FPGA is the nexus for data to and from all three USB PHYs, and can forward, alter, or reject data as desired.

Initial work focused on developing an open source USB 3.0 Verilog core which could be used to exchange large volumes of data with a monitoring computer. This work also validated the electrical characteristics of the USB 3.0 hardware, which is quite demanding due to its speed. With this earned confidence in our hardware design, we designed the USB 3.0 front end board.

When the hardware was complete, we started developing a pass-through application to prove our approach to forwarding data between USB 3.0 PHYs. Inspiration was taken from the USB 3.1 specification[10] Appendix E, which describes a "retimer" that actively retransmits data between two USB 3.0 Super Speed ports. Verilog code was derived from our USB 3.0 core to track the state of link negotiation and perform data whitening/scrambling and clock offset compensation.

We demonstrated link negotiation completing while the Daisho hardware was installed between a Linux host and a flash drive. However, mere milliseconds after link negotiation, the link consistently fails. Our current hypothesis is that clock offset compensation, which is achieved by absorbing or producing SKP (skip) symbols, is not done correctly. This is causing a loss of state in the scrambling process and places invalid data on the link.


## TURNIPSCHOOL

TURNIPSCHOOL[^11] is a radio interface concealed within the A connector of a USB cable. Consisting of a microcontroller, digital radio, and USB hub, the device provides a covert RF interface to software running on the host computer while simultaneously permitting normal operation of a downstream USB device at the other end of the cable. TURNIPSCHOOL can also be used as a radio-controlled USB device, potentially emulating other devices or implementing methods to probe the USB host.

We designed and built TURNIPSCHOOL to demonstrate that capabilities such as miniaturization of covert electronic devices are accessible to anyone, not just nation-state actors. The device was created using open source software design tools, readily available components, and hobbyist-level assembly techniques.
Solder paste stenciling with hot plate reflow was our method of PCB assembly, and we used a hand-formable silicone rubber compound to produce the overmold boot, shaped with a 3D printed mold[12]. The micro-B connector at the far end of the cable as well as the cable itself were taken from a commercial off-the-shelf cable with its A connector removed.

We demonstrated TURNIPSCHOOL operating in the 900 MHz ISM band, but the radio, based on the popular CC1111 RF transceiver, can operate on many frequencies, primarily in the 300, 400, 800, and 900 MHz bands.

During assembly, we compromised the performance of the monopole antenna by coiling it to fit within the plug body. This reduced the RF performance by approximately 10 dB compared to a straight monopole. We considered other options such as running a monopole down the cable or using a chip antenna, but all options required us to place the antenna very close to a ground plane or shield. We estimated that coiling a monopole on top of the PCB in the plug body was easiest and was likely no worse than any other antenna configuration that could be fully concealed.

We installed an open source bootloader[^13] in the microcontroller's flash memory. This allows us to modify the application firmware at any time, but a method was needed to reliably activate the bootloader. Our primary method of bootloader activation is a software trigger from the application firmware, but this method can fail if broken firmware is installed. As a more reliable secondary method, we added a magnetic sensor. When TURNIPSCHOOL is plugged in to a host computer while a magnet is held close to the plug, the bootloader is activated.


## Conclusion

All three of these tools provide platforms from which to further USB research, experimentation and security. In providing open platforms we encourage others to push these boundaries and share their findings publicly.

Our success with these projects, while still only partial, should serve to motivate others to tackle challenging hardware problems. With careful and deliberate effort and help from the growing community of hardware hackers, functional hardware was produced without the benefit of formal electrical engineering education. All of these systems were produced by a few determined people working across several time zones, and have produced valuable design documentation and code that we hope will serve as a foundation for other projects.

These projects show that the development of such tools is accessible to hobbyists and enthusiasts and does not require the resources of a large government agency.


## Acknowledgements

USBProxy has had contributions from many developers, for which we are very grateful. We would especially like to thank Adam Stasiak and Alexander Holler.

Daisho would not have been possible without the funding made available by the DARPA Cyber Fast Track program. We would like to thank our co-developers, Marshall Hecht, Benjamin Vernoux, and Mike Kershaw.

TURNIPSCHOOL is an original Great Scott Gadgets design which makes use of the software and firmware made available by RfCat[^14]. We would like to thank Atlas for the RfCat project and Karoline "Kiki der Gecko" Busse for designing our USB connector mold[^12].


## References

[1] USBProxy, https://github.com/dominicgs/USBProxy
[2] USBProxy: An Open and Affordable USB Man in the Middle Device, ShmooCon proceedings 2014
[3] Xbox 360 controller imitation, http://controllingxbox.blogspot.co.uk/
[4] Wii U controller protocol, http://www.polygon.com/2014/11/25/7287029/super-smash-bros-gc-adapter
[5] USB write blocking with USBProxy, https://www.youtube.com/watch?v=IQpPgnU-O-A
[6] FaceDancer, Travis Goodspeed, Sergey Bratus, http://goodfet.sourceforge.net/hardware/facedancer21
[7] Daisho: https://github.com/mossmann/daisho
[8] What's on the Wire? Physical Layer Tapping with Project Daisho, Black Hat USA 2013, https://media.blackhat.com/us-13/US-13-Spill-Whats-on-the-Wire-WP.pdf
[9] Cyber Fast Track, http://www.darpa.mil/OpenCatalog/CFT.html
[10] USB 3.1 Specification, http://www.usb.org/developers/docs/
[11] TURNIPSCHOOL, https://github.com/mossmann/cc11xx/tree/master/turnipschool
[12] TURNIPSCHOOL casing, https://github.com/kikithegecko/turnipschool-casing
[13] CC Bootloader, https://github.com/swift-nav/CC-Bootloader
[14] RfCat, https://bitbucket.org/atlas0fd00m/rfcat


Title: NSA Playset: USB Tools
Primary Author Name: Dominic Spill
Primary Author Email: dominicgs@gmail.com

Additional Author Name: Michael Ossmann
Additional Author Affiliation: Great Scott Gadgets
Additional Author Email: mike@ossmann.com

Additional Author Name: Jared Boone
Additional Author Affiliation: ShareBrained Technology, Inc.
Additional Author Email: jared@sharebrained.com

Keywords: USB, Packet Sniffer, Open Source, Hardware
