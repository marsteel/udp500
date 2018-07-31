# udp500
Find ISAKMP UDP500 top talker on Cisco ASR IOS XE routers with NetFlow feature. For IKE/IPsec enabled Cisco router, it might have thousands of IPsec tunnels with different peers. Some peers will send huge amount of IKE control plane packet via inbound UDP500 packets, hence the CPU% and MEMORY% is consumed. Policing such traffic is required. But neither Cisco ASR or Cisco ASA couldn't policing the traffic on a per peer IP basis. If the Cisco ASR router suffers from such problem, NetFlow need to be configured to find out the abnormal UDP500 talkers. Without dedicated NetFlow solution, a local NetFlow provided by Cisco IOS XE can be used 

All the following configurations is tested on Cisco ASR running Cisco IOS XE Software, Version 16.03.06.

Configuration

ASR#sh run | sec flow
flow record Incoming-UDP500
match transport udp destination-port
match ipv4 source address
collect counter bytes
collect counter packets
flow exporter UDP500_exporter
flow monitor UDP500_Monitor
record Incoming-UDP500
ip flow monitor UDP500_Monitor input
alias exec udp500 sh flow mon UDP500_Monitor cache filter transport udp destination-port 500 sort highest counter packet top 50

!!!under Internet-facing int G0/0/0, Apply the monitor
ip flow monitor UDP500_Monitor input

Usage

Type "udp500" on exec mode, and top 50 UDP500 talkers will be displayed. 

ASR#udp500
Processed 2764 flows
Matched 50 flows
Aggregated to 50 flows
Showing the top 50 flows

IPV4 SRC ADDR    UDP DST PORT       bytes        pkts
===============  ============  ==========  ==========
xx.56.xxx.156            500    16477820       88214
xxx.47.xx.1              500    13959920       42680
xx.78.xxx.42             500     2255632       15065
xx.0.x.66                500      927144        6935
x.107.x.198              500     1507284        5841
x.135.x.2                500     1127736        3641
x.x.4.x                  500      958728        3424
x.x.5.242                500      738648        2640


Follow up with Peer's network administrator

Check each top talker about the following behavior

o	Continuous inbound Phase 1 request via ISAKMP UDP500
	Multiple IKE/Phase1 SA are Up
	Multiple IKE/Phase1 SA are Down, and Peer is keeping requesting new IKE/Phase1,  Multiple short-lived IKE/Phase1 SA
	Show cry isa sa | sec Peer-IP
o	Continuous inbound Phase 2 request via ISAKMP UDP500
	Multiple IPsec/Phase2 SA are Up for one single ACL entry
	Multiple IPsec/Phase2 requests for non-configured ACL entry
