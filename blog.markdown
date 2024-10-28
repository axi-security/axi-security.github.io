---
layout: post
title: Blog
permalink: /123247/
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

### Read Transaction AXI

The AXI protocol is based on a handshake mechanism across distinct channels. A readtransaction involves two sequential handshakes: the *Read Address Channel Handshake* and the *Read Data Channel Handshake*.
- Read Address Channel Handshake: This initial handshake transmits the address along with essential transaction details.
- Read Data Channel Handshake: Following the address handshake, this handshake delivers the data from the specified address back to the manager.

The AXI Specification defines a strict ordering dependency between the two channels, to ensure accurate and reliable transfer. 

AXI defines many more signals , which are omitted in the figure below for simplicity. 
![Read Trans Gif](/assets/read_trans.gif)

### Write Transaction AXI

![Write Trans Gif](/assets/write_trans.gif)

## SVA
To accurately capture the behavior of a designs behavior when implemented in SystemVerilog, SystemVerilog Assertions (SVA) are employed. SVAs allow designers to specify rules that define temporal properties and constraints of the design, ensuring the protocol operates as expected and helping to detect any deviations from the intended behavior early in the verification process. SVAs allow for fine grained constraints on the design.

The inherent ambiguity in the AXI specification can lead to challenges in verifying correct operation. To analyze the protocol's security more effectively, additional properties were introduced that impose stricter constraints than those in the specification. These properties help ensure more predictable and secure interactions between components. 

```verilog
assert property (@(posedge ACLK) a -> b) 
```

The above property would encode that whenever a is high then b must be high as well. 
More information on System Verilog Assertions can be found [here](https://www.systemverilog.io/verification/sva-basics/). 

## Example Linear

### Code & Underlying Issue
Example Code from AMD Xilinx Lite Subordinate. The code below is responsible for sttoring the transmitted address and setting the subordinate driven signal ARREADY correspondingly. Things to notice in this snippet of code : 
1. ARADDR is stored into internal register axi\_araddr when S\_AXI\_ARVALID is high and axi\_arready is low. 
2. S\_AXI\_ARVALID is driven by the Manager Interface
3. S\_AXI\_ARADDR is driven by the Manager Interface

Essentially this code relies on the stability of ARADDR and specification adherence of ARVALID that states, that ARVALID must remain high until ARREADY is high. In this example we are going to focus on the property that ARVALID must remain high until ARREADY is high. 
```verilog
always @( posedge S_AXI_ACLK ) begin
    if ( S_AXI_ARESETN == 1'b0 ) begin
	      axi_arready <= 1'b0;
	      axi_araddr  <= 32'b0;
	end else if (~axi_arready && S_AXI_ARVALID) begin // CONDITION 
        axi_arready <= 1'b1;        // arready set high
        axi_araddr  <= S_AXI_ARADDR; // address latched
    end else begin 
        axi_arready <= 1'b0; // arready cleared
    end
end 
```

**Scenario 1**: In this code snippet, the subordinate depends heavily on the manager's correct operation to properly latch the address. The S\_AXI\_ARVALID signal is driven by the manager. Furthermore the specification requires both ARVALID and ARREADY signals to be high simultaneously for the address to be latched. If we assume that the manager follows this requirement and does not deassert ARVALID until both signals are high, the address will be latched before the handshake occurs.

Consider the following sequence of event:

<div class="trace_tables">
<table><thead>
  <tr>
    <th>Cycle (T)</th>
    <th>1</th>
    <th>2</th>
  </tr></thead>
<tbody>
  <tr>
    <td>ARADDR</td>
    <td>a</td>
    <td>a</td>
  </tr>
  <tr>
    <td>ARVALID</td>
    <td>1</td>
    <td>0</td>
  </tr>
  <tr>
    <td>ARREADY</td>
    <td>0</td>
    <td>1</td>
  </tr>
</tbody>
</table>
</div>


However, if ARVALID is deasserted too soon, without a handshake occuring, the address will still be latched. Additionally, we note that araddr is only cleared during a reset. This means that the latched address will remain stored internally, even if no handshake has taken place, potentially creating vulnerabilities.

**Scenario 2**: Another potential failure occurs when the manager correctly avoids prematurely deasserting ARVALID, but fails to uphold the address stability property. This property states that ARADDR must remain stable (i.e., unchanged) until the address handshake is complete.

For example, consider the following sequence of events:

<div class="trace_tables">
<table><thead>
  <tr>
    <th>Cycle (T)</th>
    <th>1</th>
    <th>2</th>
  </tr></thead>
<tbody>
  <tr>
    <td>ARADDR</td>
    <td>a</td>
    <td>b</td>
  </tr>
  <tr>
    <td>ARVALID</td>
    <td>1</td>
    <td>0</td>
  </tr>
  <tr>
    <td>ARREADY</td>
    <td>1</td>
    <td>1</td>
  </tr>
</tbody>
</table>
</div>


In this scenario, even though both ARVALID and ARREADY are high at time 2, araddr changes from a to b. According to the specification, araddr = a will be latched instead of the expected araddr = b.

While standard verification IPs can detect both of these violations since they directly contradict the specification, they may not adequately capture the security implications if such a violation occurs. Additionally, specifications that are loosely written can leave room for interpretation regarding what constitutes correct behavior. 


### Property Encoding

In order to accurately capture the behavior we need to encode the behavior. This is done via System Verilog Assertions. Our model is composed off the *base model* and the *enhanced model*. Whereas the *enhanced model* builds on the *base model*. The *base model* effectively captures the defined behavior from the specification, whereas the *enhanced model* defines identified security properties on top. **eXpect** mainly focuses on the signals that are used to constitute the handshakes.

**Explanation Property** :
- disable iff (S\_AXI\_ARVALID && axi\_arready) : when both S\_AXI\_ARVALID and axi\_arready are high the property is not checked.
- a => b : evaluation of b is one cycle after a has occurred. Thus for our a = S\_AXI\_ARESETN && S\_AXI\_ARVALID && axi\_arready and b = S\_AXI\_ARVALID we check whether S\_AXI\_ARVALID is high one cycle after S\_AXI\_ARVALID && !axi\_arready

**Example Trace Evaluation**:

Consider Scenario: 

<div class = "trace_tables">
<table><thead>
  <tr>
    <th>Cycle (t)</th>
    <th>1</th>
    <th>2</th>
    <th>3</th>
    <th>4</th>
  </tr></thead>
<tbody>
  <tr>
    <td>ARVALID</td>
    <td>0</td>
    <td>1</td>
    <td>1</td>
    <td>0</td>
  </tr>
  <tr>
    <td>ARREADY</td>
    <td>0</td>
    <td>0</td>
    <td>1</td>
    <td>0</td>
  </tr>
</tbody>
</table>
</div>

At t = 1 : a = False, thus property not triggered. At t = 2: a = True, and by looking at t = 3, we can see that b = True. At t = 3 : the property is disabled, as we do not arvalid does not need to be high at t = 4. At t = 4: a = False, thus property not evaluated. 

```verilog
assert property (@(posedge S_AXI_ACLK) disable iff (S_AXI_ARVALID && axi_arready) 
                (S_AXI_ARESETN && S_AXI_ARVALID && !axi_arready) => S_AXI_ARVALID
```


Second Example needed ? 

**Explanation Property** : 

**Example Trace Evaluation** :

```verilog 
assert property (@(posedge S_AXI_ACLK) disable iff (S_AXI_ARVALID && axi_arready) 
                (S_AXI_ARESETN) && S_AXI_ARVALID && !axi_arready) => ($past(S_AXI_ARADDR) == S_AXI_ARADDR)
```

The complete set of properties can be found [here](https://github.com/axi-security/eXpect).

### Tool

**eXpect** formally verifies any given AXI manager or subordinate implementation against it's defined properties with the help of the Questa Prop Check Tool. Questa Prop Check outputs a counterexample to the given property, or outputs the property as proven for the given implementation. **eXpect** is defined for AXI4-Lite as well as AXI4. ...  
![eXpect Tool](/assets/eXpect_Tool.png)



