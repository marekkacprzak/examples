kubectl apply -f /opt/nginx-limit.yaml
kubectl get pods

NAME                     READY   STATUS    RESTARTS   AGE
nginx-59df856cf9-rxkz4   1/1     Running   0          31s

kubectl describe node host01
  Namespace                   Name                                                 CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                                 ------------  ----------  ---------------  -------------  ---
  default                     nginx-59df856cf9-rxkz4                               250m (12%)    500m (25%)  64Mi (3%)        128Mi (6%)     56s
 
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests     Limits
  --------           --------     ------
  cpu                1140m (56%)  500m (25%)
  memory             164Mi (8%)   128Mi (6%)
  ephemeral-storage  0 (0%)       0 (0%)
  hugepages-2Mi      0 (0%)       0 (0%)

kubectl top pods

NAME                     CPU(cores)   MEMORY(bytes)
nginx-59df856cf9-rxkz4   0m           3Mi

ls -1F /sys/fs/cgroup/memory/

kubepods.slice/

 kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-59df856cf9-rxkz4   1/1     Running   0          14m

/opt/cgroup-info nginx-59df856cf9-rxkz4

sed -i 's/\r$//' /opt/cgroup-info

apt install dos2unix

dos2unix /opt/cgroup-info

/opt/cgroup-info nginx-59df856cf9-rxkz4

Container Runtime
-----------------
Pod ID: be928f1c7fd3f3aab4cf6866430b5c00f23ce9e1f11129c6adc6a757d7bc2d89
Cgroup path: /kubepods.slice/kubepods-burstable.slice/kubepods-burstable-podba260d75_3256_4778_ad3c_0f721cc7d067.slice

CPU Settings
------------
CPU Shares: 256
CPU Quota (us): 50000 per 100000

Memory Settings
---------------
Limit (bytes): 134217728

kubectl apply -f /opt/iperf-server.yaml

kubectl apply -f /opt/iperf.yaml

kubectl logs iperf

Connecting to host iperf-server, port 5201
[  5] local 172.31.239.205 port 44948 connected to 10.102.114.57 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec  86.5 MBytes   725 Mbits/sec   60    356 KBytes
[  5]   1.00-2.02   sec  84.0 MBytes   689 Mbits/sec  129    255 KBytes
[  5]   2.02-3.00   sec  54.7 MBytes   469 Mbits/sec   69    300 KBytes
[  5]   3.00-4.00   sec   107 MBytes   900 Mbits/sec  113    362 KBytes
[  5]   4.00-5.00   sec  70.4 MBytes   591 Mbits/sec  223    280 KBytes
[  5]   5.00-6.00   sec   126 MBytes  1.06 Gbits/sec  138    299 KBytes
[  5]   6.00-7.00   sec   115 MBytes   968 Mbits/sec  107    270 KBytes
[  5]   7.00-8.00   sec  76.7 MBytes   643 Mbits/sec  128    348 KBytes
[  5]   8.00-9.01   sec   106 MBytes   884 Mbits/sec  230    229 KBytes
[  5]   9.01-10.00  sec  94.5 MBytes   796 Mbits/sec    7    366 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec   921 MBytes   773 Mbits/sec  1204             sender
[  5]   0.00-10.00  sec   919 MBytes   770 Mbits/sec                  receiver

kubectl delete pod iperf

kubectl apply -f /opt/iperf-limit.yaml

kubectl logs iperf-limit

Connecting to host iperf-server, port 5201
[  5] local 172.31.239.203 port 42552 connected to 10.102.150.207 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec  22.2 MBytes   187 Mbits/sec    0   64.2 KBytes
[  5]   1.00-2.00   sec  1.25 MBytes  10.5 Mbits/sec    0    718 KBytes
[  5]   2.00-3.00   sec  0.00 Bytes  0.00 bits/sec    0    718 KBytes
[  5]   3.00-4.00   sec  0.00 Bytes  0.00 bits/sec    0    718 KBytes
[  5]   4.00-5.00   sec  0.00 Bytes  0.00 bits/sec    0    718 KBytes
[  5]   5.00-6.00   sec  0.00 Bytes  0.00 bits/sec    0    718 KBytes
[  5]   6.00-7.00   sec  0.00 Bytes  0.00 bits/sec    0    718 KBytes
[  5]   7.00-8.00   sec  0.00 Bytes  0.00 bits/sec    0    718 KBytes
[  5]   8.00-9.00   sec  0.00 Bytes  0.00 bits/sec    0    718 KBytes
[  5]   9.00-10.00  sec  0.00 Bytes  0.00 bits/sec    0    718 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  23.5 MBytes  19.7 Mbits/sec    0             sender
[  5]   0.00-10.12  sec  20.7 MBytes  17.2 Mbits/sec                  receiver

tc qdisc show

qdisc tbf 1: dev calid43b03f2e06 root refcnt 2 rate 1Mbit burst 21474835b lat 4123.2s

kubectl delete -f /opt/iperf-limit.yaml
kubectl delete -f /opt/iperf-server.yaml

kubectl create namespace sample

kubectl apply -f /opt/quota.yaml

kubectl describe namespace sample

Name:         sample
Labels:       kubernetes.io/metadata.name=sample
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:            sample-quota
  Resource         Used  Hard
  --------         ---   ---
  limits.cpu       0     2
  limits.memory    0     512Mi
  requests.cpu     0     1
  requests.memory  0     256Mi

No LimitRange resource.

kubeadm kubeconfig user --client-name=me --config /etc/kubernetes/kubeadm-init.yaml > kubeconfig

kubectl apply -f /opt/edit-bind.yaml

export KUBECONFIG=kubeconfig

kubectl delete -n sample resourcequota sample-quota
Error from server (Forbidden): resourcequotas "sample-quota" is forbidden: User "me" cannot delete resource "resourcequotas" in API group "" in the namespace "sample"

kubectl run -n sample nginx --image=nginx
Error from server (Forbidden): pods "nginx" is forbidden: failed quota: sample-quota: must specify limits.cpu for: nginx; limits.memory for: nginx; requests.cpu for: nginx; requests.memory for: nginx

kubectl apply -n sample -f /opt/sleep.yaml

kubectl get -n sample pods
NAME                    READY   STATUS    RESTARTS   AGE
sleep-7c5bb7954-m9fzf   1/1     Running   0          22s

kubectl describe namespace sample

Name:         sample
Labels:       kubernetes.io/metadata.name=sample
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:            sample-quota
  Resource         Used   Hard
  --------         ---    ---
  limits.cpu       512m   2
  limits.memory    128Mi  512Mi
  requests.cpu     250m   1
  requests.memory  64Mi   256Mi

No LimitRange resource.

kubectl scale -n sample deployment sleep --replicas=12

kubectl get -n sample pods
NAME                    READY   STATUS    RESTARTS   AGE
sleep-7c5bb7954-h6nn8   1/1     Running   0          15s
sleep-7c5bb7954-l5jsr   1/1     Running   0          15s
sleep-7c5bb7954-m9fzf   1/1     Running   0          118s

kubectl describe -n sample deployment sleep

Name:                   sleep
Namespace:              sample
Replicas:               12 desired | 3 updated | 3 total | 3 available | 9 unavailable
Conditions:
  Type             Status  Reason
  ----             ------  ------
  Progressing      True    NewReplicaSetAvailable
  Available        False   MinimumReplicasUnavailable
  ReplicaFailure   True    FailedCreate
OldReplicaSets:    <none>
NewReplicaSet:     sleep-7c5bb7954 (3/12 replicas created)

kubectl describe namespace sample

Name:         sample
Labels:       kubernetes.io/metadata.name=sample
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:            sample-quota
  Resource         Used   Hard
  --------         ---    ---
  limits.cpu       1536m  2
  limits.memory    384Mi  512Mi
  requests.cpu     750m   1
  requests.memory  192Mi  256Mi

No LimitRange resource.

