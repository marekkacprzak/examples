POD_ID=$(crictl pods --name pod1 -q)
350d436f77ebbc90b0bcf68630a21a80b7d836707dfc1ee00f79af56ef33fed3

NETNS_PATH=$(crictl inspectp $POD_ID | jq -r '.info.runtimeSpec.linux.namespaces[]|select(.type=="network").path')
/var/run/netns/cni-8963b0e9-cc9b-796e-28c6-30ca9347be18

NETNS=$(basename $NETNS_PATH)
cni-8963b0e9-cc9b-796e-28c6-30ca9347be18

ps $(ip netns pids $NETNS)
    PID TTY      STAT   TIME COMMAND
  21570 ?        Ss     0:00 /pause  <--- this process to create caliction namespace network interface
  22518 ?        Ss     0:00 sleep infinity

ip addr show | grep -B1 -A2 $NETNS
20: calice0906292e2@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-8963b0e9-cc9b-796e-28c6-30ca9347be18
    inet6 fe80::ecee:eeff:feee:eeee/64 scope link
       valid_lft forever preferred_lft forever

ip netns exec $NETNS ip addr
2: eth0@if20: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether ae:ea:57:8b:4b:73 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.31.239.204/32 scope global eth0
       valid_lft forever preferred_lft forever

ip netns exec $NETNS ip route
default via 169.254.1.1 dev eth0
169.254.1.1 dev eth0 scope link

ip netns exec $NETNS ip neigh show
169.254.1.1 dev eth0 lladdr ee:ee:ee:ee:ee:ee STALE
10.0.2.15 dev eth0 lladdr ee:ee:ee:ee:ee:ee STALE

which mean that 172.31.239.204/32 all network should be out from route table - 169.254.1.1 - but there is no ip in interface but there is in arp table
that ip 169.254.1.1 is ee:ee:ee:ee:ee:ee mac address - that on the host is calice0906292e2

kubectl apply -f /opt/two-pods.yaml
kubectl get pods -o wide
IP1=$(kubectl get po pod1 -o json | jq -r '.status.podIP')
IP2=$(kubectl get po pod2 -o json | jq -r '.status.podIP')

echo $IP1
172.31.239.204

echo $IP2
172.31.89.205

kubectl exec -it pod1 -- ping -c 1 $IP2
PING 172.31.89.205 (172.31.89.205): 56 data bytes
64 bytes from 172.31.89.205: seq=0 ttl=62 time=2.556 ms

kubectl exec -it pod1 -- ip addr
2: eth0@if20: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1450 qdisc noqueue qlen 1000
    link/ether ae:ea:57:8b:4b:73 brd ff:ff:ff:ff:ff:ff
    inet 172.31.239.204/32 scope global eth0
       valid_lft forever preferred_lft forever

kubectl exec -it pod1 -- ip route
default via 169.254.1.1 dev eth0
169.254.1.1 dev eth0 scope link

kubectl exec -it pod1 -- arp -n
? (169.254.1.1) at ee:ee:ee:ee:ee:ee [ether]  on eth0
? (10.0.2.15) at ee:ee:ee:ee:ee:ee [ether]  on eth0

tcpdump -n -w pings.pcap -i any icmp &
kubectl exec -it pod1 -- ping -c 3 $IP2
killall tcpdump
tcpdump -enr pings.pcap
21:43:41.935924  In ae:ea:57:8b:4b:73 ethertype IPv4 (0x0800), length 100: 172.31.239.204 > 172.31.89.205: ICMP echo request, id 39, seq 0, length 64
21:43:41.936086 Out 08:00:27:31:d9:0d ethertype IPv4 (0x0800), length 100: 172.31.239.204 > 172.31.89.205: ICMP echo request, id 39, seq 0, length 64
21:43:41.936824  In 08:00:27:d4:d6:87 ethertype IPv4 (0x0800), length 100: 172.31.89.205 > 172.31.239.204: ICMP echo reply, id 39, seq 0, length 64
21:43:41.936839 Out ee:ee:ee:ee:ee:ee ethertype IPv4 (0x0800), length 100: 172.31.89.205 > 172.31.239.204: ICMP echo reply, id 39, seq 0, length 64

ip a | grep -a1 08:00:27:31:d9:0d
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:31:d9:0d brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.11/24 brd 192.168.56.255 scope global enp0s8


---------------------------------------------

kubectl create -f /opt/multus-daemonset.yaml
kubectl create -f /opt/netattach.yaml
more /etc/cni/net.d/00-multus.conf
kubectl apply -f /opt/local-pods.yaml
kubectl exec -it pod1 -- ip addr
2: eth0@if18: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1450 qdisc noqueue qlen 1000
    link/ether 2e:bd:cc:17:79:1a brd ff:ff:ff:ff:ff:ff
    inet 172.31.239.204/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::2cbd:ccff:fe17:791a/64 scope link
       valid_lft forever preferred_lft forever
3: net1@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue
    link/ether 06:3b:95:45:d2:01 brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.2/24 brd 10.244.0.255 scope global net1
       valid_lft forever preferred_lft forever
kubectl exec -it pod2 -- ip addr
2: eth0@if17: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1450 qdisc noqueue qlen 1000
    link/ether fe:31:13:df:6c:91 brd ff:ff:ff:ff:ff:ff
    inet 172.31.239.203/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::fc31:13ff:fedf:6c91/64 scope link
       valid_lft forever preferred_lft forever
3: net1@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue
    link/ether 66:30:37:17:9e:70 brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.3/24 brd 10.244.0.255 scope global net1
       valid_lft forever preferred_lft forever
kubectl exec -it pod1 -- ping -c 3 10.244.0.3