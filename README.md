Introduction

Distributed Denial-of-Service attack (DDoS attack) is a cyber-attack in which the perpetrator seeks to make a machine or network resource unavailable to its intended users by temporarily or indefinitely disrupting services of a host connected to the Internet. Denial of service is typically accomplished by flooding the targeted machine or resource with superfluous requests in an attempt to overload systems and prevent some or all legitimate requests from being fulfilled.
In a DDoS attack, the incoming traffic flooding the victim originates from many different sources. This effectively makes it impossible to stop the attack simply by blocking a single source.
A DDoS attack is analogous to a group of people crowding the entry door of a shop, making it hard for legitimate customers to enter, thus disrupting trade.
The objective is to emulate a network with multiple hosts and switches, create traffic in that network, launch DDoS attack and it’s early detection so that it can be prevented.


Implementation

1.	Network Creation:
The network emulator used is Mininet for creation of network topology as shown below. Download and install Mininet from here.
Use the following command to create a network of 9 switches and 64 hosts.
sudo mn --switch ovsk --topo tree,depth=2,fanout=8 --controller=remote,ip=127.0.0.1,port=6633


2.	Network Monitoring:
POX SDN controller is used to monitor the network traffic and it’s entropy. It is present as part of Mininet. POX provides a framework for communicating with SDN (Software defined networks) switches using either the OpenFlow or OVSDB protocol. Developers can use POX to create an SDN controller using the Python programming language.
I have modified the POX controller as per the needs. Start the POX controller as follows:
cp l3_learning_edit.py <POX_dir>/pox/forwarding cp detection.py <POX_dir>/pox/forwarding
cd <POX_dir>
python pox.py pox.forwarding.l3_learning_edit
 
l3_learning_edit.py is modified to collect the statistics of the network and calculate it’s Entropy to take decision regarding DDoS prevention.

Explanation:

In DDoS, if the source addresses of incoming are spoofed, then the switch would not find a match and the packet needs to be forwarded to the controller. The collection of DDoS spoofed packets and legitimate packets can bind the controller into continuous processing, which exhausts them. Due to this, the controller is unreachable for the new incoming legitimate packets. This will bring the controller down causing loss to the SDN architecture. For a backup controller, the same challenge is to be faced.
Such kind of attacks can be detected in the early stage by monitoring few hundred of packets considering changes in entropy. The early detection of DDoS attacks stops the controller from going down. The term ‘early’ is related to tolerance level and traffic being handled by the controller. Due to this, the impact of malicious packets flooding can be controlled. Such a mechanism needs to be lightweight and high response time. The high response time saves the controller during the attack period for regaining the control by terminating the DDoS attack.

Why entropy?
The main reason for considering entropy is its ability for measuring randomness in a network. The higher the randomness the lower is the entropy and vice versa. So, whenever the entropy is less than a threshold value, we can say that a DDoS attack has occurred.

3.	Traffic Generation

Traffic generation is done with the help of Scapy. It is used for generation of packets, sniffing, scanning, forging of packet and attacking. Scapy is used for generation of UDP packets and spoofing the source IP address of the packets.
Install Scapy:
sudo apt-get install python-scapy
Run the traffic generation script from one of the host which generates random source IP and send the packets to random destinations. Observe the entropy in POX controller terminal.
cp Trafficlauch.py <mininet_dir>/custom cp attacklaunch.py <mininet_dir>/custom
Open xterm for a host by typing the following command in mininet terminal:
mininet>xterm h1
In xterm window of host h1, run the following command:
cd <mininet_dir>/custom
python launchTraffic.py –s 2 –e 65
 
 
 4.	Launch Attack
 
Now repeat above step on h1 and parallelly enter the following command to run the attack traffic from h4 and h6 xterm windows to attack on h56.
python launchAttack.py 10.0.0.56

Observing the entropy values in the POX controller. The value decreases below the threshold value (which is equal to 0.5 here) for normal traffic. Thus, we can detect the attack within the first 250 packets of malicious type of traffic attacking a host in the SDN network.
After the hosts stop sending attack packets, the switches are started again by the POX controller after shutting them down during the attack.

5.	DDoS Detection

First, make a count of 50 packets in a window and then calculate the entropy and compare it with threshold and make a count of consecutive entropy value lower than threshold. If this count reaches 5 then DDoS had occurred otherwise not.
For this, a detection script is created and some changes in the l3_learning module of the pox controller are also done so that it can detect the DDoS. These changes are explained below.
Entropy formula
Entropy is detected with the help of 2 factors:
1.	Destination IP
2.	No. of times it repeated.
Window size is kept as 50 and probability of a destination IP occurred in the window is given as pi
pi=(xi)/n	where x is no. of event in the set and n to be the window size.
 
Now,
entropy H= - sum of all (pi)log(pi)	where i is from o to n
