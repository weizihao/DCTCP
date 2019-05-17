This is an implementation of Data Center TCP (DCTCP), an enhancement to the 
TCP congestion control algorithm for data centers. The patch applies to Linux 5.0.9.
There are patches for lower version kernels on github if you need, but this version 
may be more suitable for current network environment. 

The implementation is based on the algorithm described in the paper 
"Data Center TCP (DCTCP)" (Alizadeh et al.), presented at SIGCOMM 2010. 
We refer to this paper for the details and evaluation of the algorithm. 
The paper is available online at 
http://www.stanford.edu/~alizade/Site/Publications_files/dctcp-final.pdf 

Authors:
Abdul Kabbani, Stanford University
Masato Yasuda, NEC Corp., Japan
Mohammad Alizadeh, Stanford University
------------------------------------------------------------------

[How to Apply Patch]

wget http://www.kernel.org/pub/linux/kernel/v5.x/linux-5.0.9.tar.gz
tar -xvzf linux-2.6.38.3.tar.gz
cp dctcp-5.0.9-rev1.0.0.patch linux-5.0.9
cd linux-5.0.9
patch -p1 < dctcp-5.0.9-rev1.0.0.patch

After applying patch, please compile the kernel as usual.
No special kernel configuration is necessary for DCTCP.

[How to Enable DCTCP]

DCTCP (over Reno) is enabled by the following commands.

sysctl -w net.ipv4.tcp_congestion_control=reno
sysctl -w net.ipv4.tcp_dctcp_enable=1
sysctl -w net.ipv4.tcp_ecn=1

DCTCP is enabled when kernel parameter net.ipv4.tcp_dctcp_enable is 1
(default value is 0) and net.ipv4.tcp_ecn is 1. (default value is 2).

In testing DCTCP, the DCTCP kernel must be running both on the sender and 
the receiver. In addition, ECN functionality must be enabled at the switch. 
The default ECN marking threshold for DCTCP is 20 packets (30KB) at 1Gbps, 
and 65 packets (~100KB) at 10Gbps.

[Other DCTCP parameters]

We added the following two parameters.

- net.ipv4.tcp_delayed_ack

The default value is 1 (Delayed ACK is enabled). Delayed ACK can be disabled by 
setting this to 0.

- net.ipv4.tcp_dctcp_shift_g

This is the parameter, g, for updating dctcp_alpha. The default value 
is 4 (i.e. g = 1/2^4 = 1/16).

In every RTT, dctcp_alpha is updated as follows:

dctcp_alpha = (1-g) * dctcp_alpha + g * F.
(F is the fraction of marked data during the last RTT.)
