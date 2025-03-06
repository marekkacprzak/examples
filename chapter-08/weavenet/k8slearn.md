kubectl apply -f /opt/two-pods.yaml

kubectl get pod -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
pod1   1/1     Running   0          82s   10.36.0.12   host01   <none>           <none>
pod2   1/1     Running   0          81s   10.32.0.6    host02   <none>           <none>

IP1=$(kubectl get po pod1 -o json | jq -r '.status.podIP')
IP2=$(kubectl get po pod2 -o json | jq -r '.status.podIP')
kubectl exec -it pod1 -- ip addr
45: eth0@if46: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1376 qdisc noqueue
    link/ether                                                                                                                                   brd ff:ff:ff:ff:ff:ff
    inet 10.36.0.12/12 brd 10.47.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::744a:cfff:fe65:4c8f/64 scope link
       valid_lft forever preferred_lft forever
exec -it pod1 -- ip route
default via 10.44.0.4 dev eth0
10.32.0.0/12 dev eth0 scope link  src 10.36.0.12

10.32.0.0/12 

kubectl exec -it pod1 -- ping -c 3 $IP2
PING 10.32.0.6 (10.32.0.6): 56 data bytes
64 bytes from 10.32.0.6: seq=0 ttl=64 time=13.105 ms
64 bytes from 10.32.0.6: seq=1 ttl=64 time=3.905 ms

exec -it pod1 -- arp -a
? (10.32.0.6) at 46:94:af:6e:7a:2e [ether]  on eth0
? (10.44.0.4) at 36:4b:d8:09:1f:2c [ether]  on eth0

POD_ID=$(crictl pods --name pod1 -q)
ce2eb1cd18bddf82ecf24ff9bfbfc649485e6819476a4ad00e4121e61103b9a6
NETNS_PATH=$(crictl inspectp $POD_ID | jq -r '.info.runtimeSpec.linux.namespaces[]|select(.type=="network").path')
/var/run/netns/cni-6fba1230-e8ad-6fb2-b8fc-3e30d628d329
NETNS=$(basename $NETNS_PATH)

ip addr show | grep -B1 -A2 $NETNS
46: vethweplce2eb1c@if45: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP group default
    link/ether 26:64:63:25:3c:00 brd ff:ff:ff:ff:ff:ff link-netns cni-6fba1230-e8ad-6fb2-b8fc-3e30d628d329
    inet6 fe80::2464:63ff:fe25:3c00/64 scope link
       valid_lft forever preferred_lft forever

ip addr show | grep -B1 -A2 36:4b:d8:09:1f:2c
6: weave: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UP group default qlen 1000
    link/ether 36:4b:d8:09:1f:2c brd ff:ff:ff:ff:ff:ff
    inet 10.44.0.4/12 brd 10.47.255.255 scope global weave
       valid_lft forever preferred_lft forever

tcpdump -n -w pings.pcap -i any icmp &
kubectl exec -it pod1 -- ping -c 3 $IP2
killall tcpdump

tcpdump -enr pings.pcap
24 pakiety:
17:05:37.690703   P 76:4a:cf:65:4c:8f ethertype IPv4 (0x0800), length 100: 10.36.0.12 > 10.32.0.6: ICMP echo request, id 53, seq 0, length 64
17:05:37.690740 Out 76:4a:cf:65:4c:8f ethertype IPv4 (0x0800), length 100: 10.36.0.12 > 10.32.0.6: ICMP echo request, id 53, seq 0, length 64
17:05:37.690742   P 76:4a:cf:65:4c:8f ethertype IPv4 (0x0800), length 100: 10.36.0.12 > 10.32.0.6: ICMP echo request, id 53, seq 0, length 64
17:05:37.690750 Out 76:4a:cf:65:4c:8f ethertype IPv4 (0x0800), length 100: 10.36.0.12 > 10.32.0.6: ICMP echo request, id 53, seq 0, length 64
17:05:37.692365   P 46:94:af:6e:7a:2e ethertype IPv4 (0x0800), length 100: 10.32.0.6 > 10.36.0.12: ICMP echo reply, id 53, seq 0, length 64
17:05:37.692642 Out 46:94:af:6e:7a:2e ethertype IPv4 (0x0800), length 100: 10.32.0.6 > 10.36.0.12: ICMP echo reply, id 53, seq 0, length 64
17:05:37.692644   P 46:94:af:6e:7a:2e ethertype IPv4 (0x0800), length 100: 10.32.0.6 > 10.36.0.12: ICMP echo reply, id 53, seq 0, length 64
17:05:37.692714 Out 46:94:af:6e:7a:2e ethertype IPv4 (0x0800), length 100: 10.32.0.6 > 10.36.0.12: ICMP echo reply, id 53, seq 0, length 64

tcpdump -n -w pings.pcap -i any udp &
kubectl exec -it pod1 -- ping -c 3 $IP2
killall tcpdump
tcpdump -enr pings.pcap -T vxlan | grep -B 1 ICMP
reading from file pings.pcap, link-type LINUX_SLL (Linux cooked v1)
17:08:18.981700 Out 08:00:27:94:42:a1 ethertype IPv4 (0x0800), length 150: 192.168.56.11.57074 > 192.168.56.12.6784: VXLAN, flags [I] (0x08), vni 2959746
76:4a:cf:65:4c:8f > 46:94:af:6e:7a:2e, ethertype IPv4 (0x0800), length 98: 10.36.0.12 > 10.32.0.6: ICMP echo request, id 59, seq 0, length 64
17:08:18.983807  In 08:00:27:77:30:ee ethertype IPv4 (0x0800), length 150: 192.168.56.12.44678 > 192.168.56.11.6784: VXLAN, flags [I] (0x08), vni 9970386
46:94:af:6e:7a:2e > 76:4a:cf:65:4c:8f, ethertype IPv4 (0x0800), length 98: 10.32.0.6 > 10.36.0.12: ICMP echo reply, id 59, seq 0, length 64

