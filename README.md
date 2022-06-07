# IP-Routing

Even when I am writing this blog, we all are communicating with the help of machines. I am not talking to you ! Its our electronic device that is communicating.

So it is very important that we learn on how two machines communicates each other.

One of the basic thing that we must understand is on how a machine knows where and how to send the packet to reach its final destination. Here I will try to explain how IP routing happens. That is, how a machine determines where to sent the packet.

Let’s consider routing table of a simple linux machine. Assume the machine is inside a subnetwork `10.68.40.0/24`. It will look somewhat like below.

```
Destination     Gateway         Genmask         Flags   MSS Window irtt Iface
0.0.0.0         10.68.40.1      0.0.0.0         UG        0 0          0 ens4
10.68.40.1      0.0.0.0         255.255.255.0 UH        0 0          0 ens4
```

In subnetwork `10.68.40.0/24` , the IP `10.68.40.1` is assigned to default gateway/router. Router is a device which is used to connect between two networks.

Now, let’s assume this machine want’s to talk to some other machine. Let’s see how kernel decides the path based on the routing table.


## Case1: A machine in another network

Let’s take a random IP `19.45.65.75`. So the kernel check which rule in the routing table matches the best. Here It doesn’t match anything. So it takes the first rule where destination is `0.0.0.0` which is the default rule.

```
Destination     Gateway         Genmask
0.0.0.0         10.68.40.1      0.0.0.0
```

This means the next hop of the packet will be the default gateway `10.68.40.1` through interface ens4 .

Now, the source address of the packet is set as interface IP and destination IP as the default gateway.

To send the packet to the gateway `10.68.40.1` linux has to figure out the MAC address of the gateway. Linux broadcasts ARP request within the internal network asking who has `10.68.40.1`. The owner of the IP sends an ARP response which is cached by the kernel and the kernel sends the packet to the gateway by setting Source mac address as mac address of ens4 and destination mac address of `10.68.40.1`. (Source — https://linkedin.github.io/school-of-sre/level101/linux_networking/ipr/)


## Case2: A machine in same network

Now let’s see what happens when it wants to communicate with a machine having IP `10.68.40.15`. Here the kernel understands that the destination IP is in the same network from the Genmask , which is the subnetmask of the network.

```
Destination     Gateway         Genmask 
10.68.40.1      0.0.0.0         255.255.255.0
```

According to this rule there is no Gateway here. So linux can just forward the packet with the help of ARP.

<img width="1023" alt="Screenshot 2022-06-07 at 23 08 38" src="https://user-images.githubusercontent.com/37524392/172448348-1e908a15-4f8b-4b22-8a0b-8c34cc7a77fd.png">

## CONCLUSION

This is a basic concept on how routing happens in a linux machine. I have tried to keep it as simple as possible. Share your comments.

Hope this helps!


