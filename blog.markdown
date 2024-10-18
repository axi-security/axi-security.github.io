---
layout: post
title: Blog
permalink: /blog/
---

## AXI Background
The Arm Advanced eXtensible Interface (AXI) protocol is a speci-
fication for system-on-chip (SoC) communication . It consists
of interfaces such as AXI-Lite and AXI-Full and is a part of the
Advanced Microcontroller Bus Architecture (AMBA) specification
suite. AXI connects Intellectual Property (IP) cores using a man-
ager/subordinate model,. It uses handshakes to coordinate
communication via read-and-write transactions.
AXI4 employs five distinct channels
allowing for parallel read and write transactions. The address channels convey both address and protection
information for both transactions through the read and write ad-
dress channel. In read transactions, data from memory or registers
are transmitted back to the manager through the read data channel.
In write transactions, the write data channel transports the data to
be written to the target address. Additionally, a dedicated acknowl-
edgment channel for write transactions transfers acknowledgment
responses from the subordinate to the manager. We reason about
all signals in AXI-Lite.

HERE COMES ANIMATION WITH HIGHLEVEL MANAGER SUBORDINATE AND THEN LOW LEVEL SIGNAL NEXT TO IT

## SVA
To accurately capture the behavior of a designs behavior when implemented in SystemVerilog, SystemVerilog Assertions (SVA) are employed. SVAs allow designers to specify rules that define temporal properties and constraints of the design, ensuring the protocol operates as expected and helping to detect any deviations from the intended behavior early in the verification process. SVAs allow for fine grained constraints on the design ... what else to say ?

HERE SMALL EXAMPLE OF HOW SVAs work wnad what can be expressed

The inherent ambiguity in the AXI specification can lead to challenges in verifying correct operation. To analyze the protocol's security more effectively, additional properties were introduced that impose stricter constraints than those in the specification. These properties help ensure more predictable and secure interactions between components. ...

## Example Linear

### Code & Underlying Issue

### Property Encoding


