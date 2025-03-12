kubectl apply -f /opt/ipf-server.yaml

kubectl get pods -o wide

NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
iperf-server-657c4c89fc-lwwpz   1/1     Running   0          42s   172.31.89.203    host02   <none>           <none>
iperf-server-657c4c89fc-thgn8   1/1     Running   0          42s   172.31.239.202   host01   <none>           <none>
iperf-server-657c4c89fc-z8chx   1/1     Running   0          42s   172.31.25.212    host03   <none>           <none>

kubectl apply -f /opt/ipf-svc.yaml

kubectl get ep iperf-server
NAME           ENDPOINTS                                                   AGE
iperf-server   172.31.239.202:5201,172.31.25.212:5201,172.31.89.203:5201   17s

kubectl apply -f /opt/ipf-client.yaml

kubectl get pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP               NODE     NOMINATED NODE   READINESS GATES
iperf-f85fd9456-8s4lf           1/1     Running   0          6s      172.31.89.204    host02   <none>           <none>
iperf-f85fd9456-nz8x9           1/1     Running   0          6s      172.31.25.213    host03   <none>           <none>
iperf-f85fd9456-xnk7w           1/1     Running   0          6s      172.31.239.203   host01   <none>           <none>
iperf-server-657c4c89fc-lwwpz   1/1     Running   0          3m34s   172.31.89.203    host02   <none>           <none>
iperf-server-657c4c89fc-thgn8   1/1     Running   0          3m34s   172.31.239.202   host01   <none>           <none>
iperf-server-657c4c89fc-z8chx   1/1     Running   0          3m34s   172.31.25.212    host03   <none>           <none>

 kubectl get pods -o wide
NAME                            READY   STATUS             RESTARTS      AGE     IP               NODE     NOMINATED NODE   READINESS GATES
iperf-f85fd9456-8s4lf           1/1     Running            1 (34s ago)   2m37s   172.31.89.204    host02   <none>           <none>
iperf-f85fd9456-nz8x9           1/1     Running            1 (34s ago)   2m37s   172.31.25.213    host03   <none>           <none>
iperf-f85fd9456-xnk7w           0/1     CrashLoopBackOff   2 (21s ago)   2m37s   172.31.239.203   host01   <none>           <none>

iptables-save | grep iperf-server

-A KUBE-SVC-KN2SIRYEH2IFQNHK ! -s 172.31.0.0/16 -d 10.97.66.77/32 -p tcp -m comment --comment "default/iperf-server cluster IP" -m tcp --dport 5201 -j KUBE-MARK-MASQ
-A KUBE-SVC-KN2SIRYEH2IFQNHK -m comment --comment "default/iperf-server -> 172.31.239.202:5201" -m statistic --mode random --probability 0.33333333349 -j KUBE-SEP-ZJYHJALL7BKIQO52
-A KUBE-SVC-KN2SIRYEH2IFQNHK -m comment --comment "default/iperf-server -> 172.31.25.212:5201" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-MLOJ3P66CAGJ7FQ4
-A KUBE-SVC-KN2SIRYEH2IFQNHK -m comment --comment "default/iperf-server -> 172.31.89.203:5201" -j KUBE-SEP-XXXQMXOMMZ6MQH27

kubectl patch svc iperf-server -p '{"spec":{"internalTrafficPolicy":"Local"}}'

iptables-save | grep iperf-server

-A KUBE-SVL-KN2SIRYEH2IFQNHK ! -s 172.31.0.0/16 -d 10.97.66.77/32 -p tcp -m comment --comment "default/iperf-server cluster IP" -m tcp --dport 5201 -j KUBE-MARK-MASQ
-A KUBE-SVL-KN2SIRYEH2IFQNHK -m comment --comment "default/iperf-server -> 172.31.239.202:5201" -j KUBE-SEP-ZJYHJALL7BKIQO52

kubectl get pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
iperf-f85fd9456-h2s9c           1/1     Running   0          32s   172.31.239.204   host01   <none>           <none>
iperf-f85fd9456-pgw79           1/1     Running   0          32s   172.31.25.214    host03   <none>           <none>
iperf-f85fd9456-sf5m4           1/1     Running   0          32s   172.31.89.205    host02   <none>           <none>

kubectl logs iperf-f85fd9456-h2s9c
Connecting to host iperf-server, port 5201
[  5] local 172.31.239.204 port 37092 connected to 10.97.66.77 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec  2.22 GBytes  19.0 Gbits/sec  264   6.07 MBytes
[  5]   1.00-2.00   sec  2.33 GBytes  20.0 Gbits/sec  287   4.25 MBytes
[  5]   2.00-3.00   sec  2.55 GBytes  21.9 Gbits/sec  2132    932 KBytes
[  5]   3.00-4.00   sec  2.60 GBytes  22.3 Gbits/sec  1734    800 KBytes
[  5]   4.00-5.00   sec  1.89 GBytes  16.2 Gbits/sec  694   1.01 MBytes
[  5]   5.00-6.00   sec  2.38 GBytes  20.4 Gbits/sec  554    841 KBytes
[  5]   6.00-7.00   sec  2.47 GBytes  21.2 Gbits/sec  1171    564 KBytes
[  5]   7.00-8.00   sec  2.60 GBytes  22.3 Gbits/sec  1960    625 KBytes
[  5]   8.00-9.00   sec  2.64 GBytes  22.7 Gbits/sec  998    609 KBytes
[  5]   9.00-10.00  sec  2.20 GBytes  18.9 Gbits/sec  2098   1.24 MBytes

kubectl delete svc/iperf-server deploy/iperf deploy/iperf-server

more /opt/add-hw.sh

#!/bin/bash
conf=/etc/kubernetes/admin.conf
cert=/etc/kubernetes/pki/admin.crt
key=/etc/kubernetes/pki/admin.key
ca=/etc/kubernetes/pki/ca.crt
grep client-key-data $conf | cut -d" " -f 6 | base64 -d > $key
grep client-cert $conf | cut -d" " -f 6 | base64 -d > $cert
patch='
[
  {
    "op": "add",
    "path": "/status/capacity/bookofkubernetes.com~1special-hw",
    "value": "3"
  }
]
'
curl --cacert $ca --cert $cert --key $key \
  -H "Content-Type: application/json-patch+json" \
  -X PATCH -d "$patch" \
  https://192.168.61.10:6443/api/v1/nodes/host02/status
echo ""

sed -i 's/\r$//'

/opt/add-hw.sh

  "status": {
    "capacity": {
      "bookofkubernetes.com/special-hw": "3",
      "cpu": "4",
      "ephemeral-storage": "40581564Ki",
      "hugepages-2Mi": "0",
      "memory": "7990752Ki",
      "pods": "110"
    },

kubectl get node host02 -o json | jq .status.capacity

{
  "bookofkubernetes.com/special-hw": "3",
  "cpu": "4",
  "ephemeral-storage": "40581564Ki",
  "hugepages-2Mi": "0",
  "memory": "7990752Ki",
  "pods": "110"
}

kubectl apply -f /opt/hw.yaml

kubectl get po -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP              NODE     NOMINATED NODE   READINESS GATES
sleep   1/1     Running   0          15s   172.31.89.206   host02   <none>           <none>

kubectl describe node host02
Name:               host02

Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource                         Requests     Limits
  --------                         --------     ------
  cpu                              1130m (28%)  0 (0%)
  memory                           100Mi (1%)   0 (0%)
  ephemeral-storage                0 (0%)       0 (0%)
  hugepages-2Mi                    0 (0%)       0 (0%)
  bookofkubernetes.com/special-hw  1            1

kubectl apply -f /opt/hw3.yaml

kubectl get po -o wide
NAME     READY   STATUS    RESTARTS   AGE    IP              NODE     NOMINATED NODE   READINESS GATES
sleep    1/1     Running   0          5m9s   172.31.89.206   host02   <none>           <none>
sleep3   0/1     Pending   0          13s    <none>          <none>   <none>           <none>

kubectl delete po sleep

kubectl get po -o wide
NAME     READY   STATUS    RESTARTS   AGE   IP              NODE     NOMINATED NODE   READINESS GATES
sleep3   1/1     Running   0          79s   172.31.89.207   host02   <none>           <none>

Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource                         Requests     Limits
  --------                         --------     ------
  cpu                              1130m (28%)  0 (0%)
  memory                           100Mi (1%)   0 (0%)
  ephemeral-storage                0 (0%)       0 (0%)
  hugepages-2Mi                    0 (0%)       0 (0%)
  bookofkubernetes.com/special-hw  3            3

