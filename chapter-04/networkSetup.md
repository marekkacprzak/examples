eth0 172.19.3.15
cni0 10.22.0.1

NETNS_PATH=$(crictl inspectp $B1P_ID | jq -r '.info.runtimeSpec.linux.namespaces[]|select(.type=="network").path')
NETNS=$(basename $NETNS_PATH)
ps --pid $(ip netns pids $NETNS)
ip netns exec $NETNS ip addr   # Show interface busy box

busybox 10.22.0.5 

-------------------------------
ip route
default via 172.19.0.1 dev eth0 proto dhcp src 172.19.3.15 metric 100 
10.22.0.0/16 dev cni0 proto kernel scope link src 10.22.0.1 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
172.19.0.0/20 dev eth0 proto kernel scope link src 172.19.3.15 metric 100 

Tworzenie nowej przestrzeni sieciowej
ip netns add myns
ip netns exec myns ip addr   # wyswietla interfejsy sieciowe w tej przestrzeni

1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

wlaczamy:

ip netns exec myns ip link set dev lo up

    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host proto kernel_lo 
       valid_lft forever preferred_lft forever

ip netns exec myns ping -c 1 127.0.0.1PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.

64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.027 ms
--- 127.0.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.027/0.027/0.027/0.000 ms

laczymy virtualny interfejs z hostem

ip link add myveth-host type veth peer myveth-myns netns myns

ip addr

 myveth-host@if2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 32:fc:e4:87:91:15 brd ff:ff:ff:ff:ff:ff link-netns myns

ip netns exec myns ip addr # myns interfejsy

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host proto kernel_lo 
       valid_lft forever preferred_lft forever
2: myveth-myns@if8: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 66:3e:04:1e:29:ef brd ff:ff:ff:ff:ff:ff link-netnsid 0

przypisujemy ip

ip netns exec myns ip addr add 10.22.0.254/16 dev myveth-myns
ip netns exec myns ip link set dev myveth-myns up
ip link set dev myveth-host up

ip netns exec myns ip addr

myveth-myns@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 66:3e:04:1e:29:ef brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.22.0.254/16 scope global myveth-myns
       valid_lft forever preferred_lft forever
    inet6 fe80::643e:4ff:fe1e:29ef/64 scope link proto kernel_ll 
       valid_lft forever preferred_lft forever


ip netns exec myns ping -c 1 10.22.0.254

PING 10.22.0.254 (10.22.0.254) 56(84) bytes of data.
64 bytes from 10.22.0.254: icmp_seq=1 ttl=64 time=0.055 ms
--- 10.22.0.254 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.055/0.055/0.055/0.000 ms

ping -c 1 10.22.0.253 # host go nie widzi

PING 10.22.0.253 (10.22.0.253) 56(84) bytes of data.
--- 10.22.0.253 ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms

dodawanie interfejsu do mostu

brctl show

bridge name	bridge id		STP enabled	interfaces
cni0		8000.aefa5655bee2	no		veth473864d9
							veth7fd5f253
							vethf4125c94
docker0		8000.024211602396	no

brctl addif cni0 myveth-host

bridge name	bridge id		STP enabled	interfaces
cni0		8000.aefa5655bee2	no		myveth-host

ping -c 1 10.22.0.254

PING 10.22.0.254 (10.22.0.254) 56(84) bytes of data.
64 bytes from 10.22.0.254: icmp_seq=1 ttl=64 time=0.095 ms
--- 10.22.0.254 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.095/0.095/0.095/0.000 ms

śledzenie ruchu

ping 10.22.0.254 >/dev/null 2>&1 &
timeout 1s tcpdump -i any -n icmp

listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
00:26:24.263121 cni0  Out IP 10.22.0.1 > 10.22.0.254: ICMP echo request, id 8825, seq 3, length 64
00:26:24.263135 myveth-host Out IP 10.22.0.1 > 10.22.0.254: ICMP echo request, id 8825, seq 3, length 64
00:26:24.263159 myveth-host P   IP 10.22.0.254 > 10.22.0.1: ICMP echo reply, id 8825, seq 3, length 64
00:26:24.263159 cni0  In  IP 10.22.0.254 > 10.22.0.1: ICMP echo reply, id 8825, seq 3, length 64

4 packets captured
4 packets received by filter
0 packets dropped by kernel

killall ping

ip route

default via 172.19.0.1 dev eth0 proto dhcp src 172.19.3.15 metric 100 
10.22.0.0/16 dev cni0 proto kernel scope link src 10.22.0.1 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
172.19.0.0/20 dev eth0 proto kernel scope link src 172.19.3.15 metric 100 

ip netns exec myns ping -c 1 172.19.3.15

ping: connect: Network is unreachable

ip netns exec myns ip route add default via 10.22.0.1

ip netns exec myns ping -c 1 172.19.3.15

PING 172.19.3.15 (172.19.3.15) 56(84) bytes of data.
64 bytes from 172.19.3.15: icmp_seq=1 ttl=64 time=0.081 ms
--- 172.19.3.15 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.081/0.081/0.081/0.000 ms

pingowanie innego hosta z poza main hosta (maskarada)

ip netns exec myns ping -c 1 172.19.3.14

PING 172.19.3.14 (172.19.3.14) 56(84) bytes of data.
--- 172.19.3.14 ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms

iptables -t nat -n -L

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
MASQUERADE  0    --  172.17.0.0/16        0.0.0.0/0           
CNI-f9f9e656e1fc70f64d2bc16d  0    --  10.22.0.3            0.0.0.0/0            /* name: "bridge" id: "f915e9c075f8dcfd5eac27ab43bf9d63fe0c8b28127cf874b8eaeaa1189e175e" */
CNI-1d663699cadca2e1c56bdf7a  0    --  10.22.0.4            0.0.0.0/0            /* name: "bridge" id: "9102e488f66b6cda23120d599341808f765fca657347011b0909e92a4c4bdba6" */
CNI-0040247445e1c1f2e908c3d3  0    --  10.22.0.5            0.0.0.0/0            /* name: "bridge" id: "700a3aad057978a893432b37f3f12a0d3a9b9a31377edb9ac3b250fded9114a7" */

Chain CNI-0040247445e1c1f2e908c3d3 (1 references)
target     prot opt source               destination         
ACCEPT     0    --  0.0.0.0/0            10.22.0.0/16         /* name: "bridge" id: "700a3aad057978a893432b37f3f12a0d3a9b9a31377edb9ac3b250fded9114a7" */
MASQUERADE  0    --  0.0.0.0/0           !224.0.0.0/4          /* name: "bridge" id: "700a3aad057978a893432b37f3f12a0d3a9b9a31377edb9ac3b250fded9114a7" */

tworzymy nowy wpis lancucha w iptable

iptables -t nat -N chain-myns

iptables -t nat -A chain-myns -d 10.22.0.0/16 -j ACCEPT

iptables -t nat -A chain-myns ! -d 224.0.0.0/4 -j MASQUERADE

i nakazujemy stosowanie tego lancucha do obslugi ruchu sieciowego pochodzacego z 10.22.9.254

iptables -t nat -A POSTROUTING -s 10.22.0.254 -j chain-myns

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination            
chain-myns  0    --  10.22.0.254          0.0.0.0/0  

Chain chain-myns (1 references)
target     prot opt source               destination         
ACCEPT     0    --  0.0.0.0/0            10.22.0.0/16        
MASQUERADE  0    --  0.0.0.0/0           !224.0.0.0/4

ip netns exec myns ping -c 1 172.19.3.14
PING 172.19.3.14 (172.19.3.14) 56(84) bytes of data.
