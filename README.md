# Openwrt with Vlans for home networking

How I setup my in-home network with VLans using OpenWRT

## Introduction

As a professional in the devops and software development fields, I
spent a fair amount of my own time keeping up to date with the
developments in the field. I also enjoy hacking around with systems.

As such, I have spent a lot of time following the OpenWRT router
project. This project provides open source firmware for many varieties
of router. I have my home configured with a couple of OpenWRT
installations which allow me to do some more sophisticated than normal
network configurations.

Several of my friends and colleages have asked me about my
configuration, so I have documented it here. Please note: The usual
caveat about modifying your own hardware applies - this is an example
and should not be used verbatim. I can not be held responsible if you
brick your router or lock yourself out!

If you are not interested in advanced home networking, or you lack
computer skills this guide probably isn't for you.

## What is OpenWRT?

[OpenWRT](https://openwrt.org/) is a suite of tooling and packages
which allow you to replace stock firmware on a commerical home or soho
router with a customized version. This is a very good thing if you are
a profession in the software or IT space, or just have an interest in
leveraging more sophisticated firmware. It is also (despite recently
found issues) widely regarded as being more secure than default vendor
firmware as that has been plagued by backdoors, default credentials,
out of date packages with known vulnerabilities etc.

OpenWRT uses a Linux kernel and a userspace, along with a highly
sophisticated "buildroot" toolchain which provides many hundreds of
pre-packaged applications. Part of the toolchain is an "imagebuilder"
tool which allows you to easily make customized images for your
hardware.

OpenWRT supports an optional GUI called "LUCI" which although not
required, makes it somewhat easier to understand complex setups.

## What is a VLan and why do I care?

A VLan is a Virtual Network (Software defined network) running on top
of your physical network layer. VLan packets are normal Internet
Protocol (IP) packets with an additional field in the header
containing a VLan number. VLans are isolated from each other such that
packets on VLan 1 will not be seen by VLan 2. See
[VLans](https://en.wikipedia.org/wiki/Virtual_LAN) for more
information.

This allows, for example, separation of my personal and work computers
by putting them into different VLans. The VLan containing my work
computer is allowed to reach out to the internet but not to open a
connection to my personal PC, and vice versa. In the event that one of
them became infected by a virus, then the other would be isolated from
it thus limiting the "spash damage" of the infection.

## What is a VPN?

A VPN is a different form of Virtual Private Network. Like a VLan it
is a software defined overlay network, but unlike VLans it uses
encyption to traverse untrusted 3rd party networks. VPNs are often
used to connect a business computer from your home to office, or
between geographically distributed sites over the internet.

## How is my network configured?

I am lucky enough to have a Google Fiber 1G up/down symmetrical fiber
connection. This connection enters my house as fiber and is
immediately converted to 1G ethernet via a dumb fiber-to-ethernet
converter. One advantage of that is that I can bin the Google head
unit/router/wifi and replace it with my own. So that is what I did.

### Firewall/Gateway router

I used a SFF X86 PC similar to this one:

[Protectli Vault 4 Port, Firewall Micro Appliance/Mini PC - Intel Quad Core, AES-NI, 8GB RAM, 120GB mSATA SSD
by protectli](https://www.amazon.com/dp/B07G9NHRGQ/ref=cm_sw_em_r_mt_dp_U_6MZMEbBEDB98D)

These are sold barebones (no RAM, SSD, or software), populated but
with software, or often with pfSense. pfSense is an alternative
firmware for customized firewalls, and widely regarded as easier to
use than OpenWRT. As, however, I was partly using this as a learning
exercise I wanted to use the less friendly version. I also wanted to
use a consistent router OS across my lan (more on this later).

If you buy one, make sure it supports the AWS-NI instructions as they
help accelerate some functions like VPNs.

Installing OpenWRT on this box is trivial, boot the box off a USB
stick with linux on, download the x86-64 image from OpenWRT, and image
it onto the internal drive. Remove the USB stick and
reboot. Conveniently, these SFF boxes have a VGA out so you can
connect a screen and troubleshoot if anything goes wrong. More details
[from
OpenWRT](https://openwrt.org/docs/guide-user/installation/openwrt_x86)

If you have a large SSD, you may want to consider partitioning it or
expanding the file system to match.

In order to have this work with the Google fiber connection, Google
needs two "magic" numbers setting on the port connected to their
service.

VLAN = 2
Ethernet QoS = 3

These two magic numbers need to be set in OpenWRT, and this is where
the first part of customization begins (see below).

### VLan capable managed switches

The next component in my home LAN is a 1G x 8 managed switch. Mine are similar to these

[NETGEAR 8-Port Gigabit Smart Managed Plus Switch (GS308E)
by Amazon.com](https://www.amazon.com/dp/B07PLFCQVK/ref=cm_sw_em_r_mt_dp_U_B0ZMEb3T0PQNG)

The important characteristics to note are the switch MUST be "managed"
and MUST support "Basic VLAN & QoS"

I have two of these, and I will explain the configuration below.

### WiFi access point and Router

The last component in my home network is a WiFi router. Note that the
SFF PC does not have WiFI capability. Some do, but don't do it as well
as commercial ones. I chose my home router very carefully as I wanted
to ensure that all the components are well supported by OpenWRT.

[NETGEAR Nighthawk X4S Smart WiFi Router (R7800) - AC2600 Wireless
Speed (up to 2600 Mbps) | Up to 2500 sq ft Coverage & 45 Devices | 4 x
1G Ethernet, 2 x 3.0 USB, and 1 x eSATA
ports](https://www.amazon.com/dp/B0192911RA/ref=cm_sw_em_r_mt_dp_U_i6ZMEbBK613V5)

The first thing I did after unboxing and powering on this router is
use the built-in firmware upgrade function to replace the firmware
with OpenWRT. This was an extremely easy and quick process. As a
bonus, I can use one of the onboard USB ports to power the
fiber-to-ethernet converter and get rid of a wall-wart.







