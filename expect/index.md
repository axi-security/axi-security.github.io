---
layout: page
title: eXpect
permalink: /expect/
---

# eXpect: On the Security Implications of Violations in AXI Implementations

![Logo](expect_logo.png){: .logo }


## Abstract

The Arm Advanced eXtensible Interface (AXI) protocol is a widely used on-chip interconnect for processors, accelerators, memories, and other IP cores. Any bugs in AXI implementations pose a security risk to the chip’s correctness. Buggy or non-compliant third-party IPs can exploit AXI implementation bugs to bypass the security mechanisms of the entire system. Identifying AXI implementation bugs is challenging because incomplete specifications allow room for implementation-specific behavior in high-performance designs.

**eXpect** is a systematic approach for analyzing AXI implementations to detect functional and security violations. We used EXPECT to test seven implementations of varying complexity, including those from AMD Xilinx and RISC-V PULP. Through this analysis, we identified 135 property violations. From these, we sampled 10 violations to demonstrate seven exploits, showing that attackers can use these bugs to compromise victim IPs. Our exploits achieved outcomes such as using stale data, skipping reads and writes, leaking intermediate data, and reading/writing attacker-controlled data to attacker-controlled addresses.

We evaluated these exploits in realistic scenarios deployed on FPGA, revealing that AMD Xilinx protocol checker IPs missed 5 out of 7 exploits.

## Authors

- [Melisande Zonta-Roudes](https://melisandezonta.com)
- Andres Meza
- Nora Hinderling
- [Lucas Deutschmann](https://eit.rptu.de/fgs/eis/people/deutschmann)
- [Francesco Restuccia](https://frestucc.github.io)
- [Ryan Kastner](https://kastner.ucsd.edu/ryan)
- [Shweta Shinde](https://n.ethz.ch/~sshivaji)

## Paper

You can view the full paper [here](expect_iccad24.pdf).


