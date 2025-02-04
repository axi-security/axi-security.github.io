---
title: "XRAY: Detecting and Exploiting Vulnerabilities in Arm AXI Interconnects"
github_repo: "https://github.com/axi-security/xray/" # Add the link to your GitHub repo here
pdf_link: "/assets/expect_iccad24.pdf" # Link to your paper PDF
citation: "Zonta-Roudes, Hinderling, N., Shinde, S. 'Xray: Detecting and Exploiting Vulnerabilities in Arm AXI Interconnects' DATE 2025."
permalink: /xray/
layout: xray
content_type: abstract
---
## Abstract

The Arm AMBA Advanced eXtensible Interface (AXI) interconnect is a critical IP in FPGA-based designs. While AXI and interconnect designs are primarily optimized for performance, their security requires closer investigation—any bugs in these components can potentially compromise critical IPs like processing systems and memory. 

To this end, **Xray** systematically analyzes AXI interconnects. Specifically, it treats the AXI interconnect as a transaction processing block that is expected to adhere to certain properties (e.g., bus and data isolation, progress). Then, **Xray** employs a traffic generator that creates transaction workloads with the aim of triggering violations in the AXI interconnects. As the last piece of the puzzle, **Xray** checkers automatically flag transaction traces are either compliant, errors, or warnings. 

Put together, **Xray** comprises 13 properties, has been tested on 7 interconnects, identifies 41 violations corresponding to 41 vulnerabilities. When compared to existing approaches such as verification IPs (VIPs) and protocol checkers from commercial tools, **Xray** identifies 19 known and 22 new violations. We show the security impact of **Xray** by sampling 5 XRAY violations to construct 3 proof-of-concept exploits on realistic scenarios deployed on FPGA to leak intermediate data, drop transactions, and corrupt memory. 


