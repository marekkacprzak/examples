kubectl apply -f /opt/best-effort.yaml

kubectl get po best-effort -o json | jq .status.qosClass
"BestEffort"

sed -i 's/\r$//' /opt/cgroup-info

/opt/cgroup-info best-effort

Container Runtime
-----------------
Pod ID: 6dbaf687cf95bdd31d61208d8a4d201a42dd47efa1ff1961889dd24d15d8fa27
Cgroup path: /kubepods.slice/kubepods-besteffort.slice/kubepods-besteffort-podb7ebe824_8621_4b7d_bef6_f45d59228bfb.slice

CPU Settings
------------
CPU Shares: 2
CPU Quota (us): -1 per 100000

Memory Settings
---------------
Limit (bytes): 9223372036854771712

cat /sys/fs/cgroup/cpu/kubepods.slice/kubepods-besteffort.slice/cpu.shares
2
cat /sys/fs/cgroup/cpu/kubepods.slice/kubepods-burstable.slice/cpu.shares
1157

kubectl apply -f /opt/burstable.yaml

kubectl get po burstable -o json | jq .status.qosClass
"Burstable"

/opt/cgroup-info burstable

Container Runtime
-----------------
Pod ID: ea95ab723e18db09b50babbec452fe376584b084f278db69436ca1f9599caa97
Cgroup path: /kubepods.slice/kubepods-burstable.slice/kubepods-burstable-pod705371a5_7c30_412a_ac2f_743876d50f97.slice

CPU Settings
------------
CPU Shares: 51
CPU Quota (us): 10000 per 100000

Memory Settings
---------------
Limit (bytes): 134217728

cat /sys/fs/cgroup/cpu/kubepods.slice/kubepods-burstable.slice/cpu.shares
1208
cat /sys/fs/cgroup/cpu/kubepods.slice/kubepods-besteffort.slice/cpu.shares
2

kubectl apply -f /opt/guaranteed.yaml

kubectl get po guaranteed -o json | jq .status.qosClass
"Guaranteed"

/opt/cgroup-info guaranteed

Container Runtime
-----------------
Pod ID: 4d8d60621f23e50b4d91a2d5bcdb3f2abb10c60d270beafdabfcd09aac432fac
Cgroup path: /kubepods.slice/kubepods-pod19746ee6_360e_4745_b1cb_f699278d747f.slice

CPU Settings
------------
CPU Shares: 51
CPU Quota (us): 5000 per 100000

Memory Settings
---------------
Limit (bytes): 67108864

sed -i 's/\r$//' /opt/oom-info

/opt/oom-info best-effort
OOM Score Adjustment: 1000
/opt/oom-info burstable
OOM Score Adjustment: 992
/opt/oom-info guaranteed
OOM Score Adjustment: -997

kubectl delete po/best-effort po/burstable po/guaranteed

kubectl apply -f /opt/essential.yaml

kubectl apply -f /opt/lots.yaml

kubectl describe deploy lots

Replicas:               1000 desired | 1000 updated | 1000 total | 31 available | 969 unavailable

kubectl describe node host01

Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests     Limits
  --------           --------     ------
  cpu                3880m (97%)  2750m (68%)
  memory             804Mi (10%)  704Mi (9%)
  ephemeral-storage  0 (0%)       0 (0%)
  hugepages-2Mi      0 (0%)       0 (0%)

kubectl get po | grep -c Pending
969

kubectl apply -f /opt/needed.yaml

kubectl get po needed
NAME     READY   STATUS    RESTARTS   AGE
needed   1/1     Running   0          41s

kubectl get po | grep -c Pending
970

