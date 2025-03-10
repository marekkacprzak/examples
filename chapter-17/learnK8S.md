kubectl get crds

NAME                                                  CREATED AT
clusterinformations.crd.projectcalico.org             2025-03-10T19:36:27Z
installations.operator.tigera.io                      2025-03-10T19:36:29Z
volumes.longhorn.io                                   2025-03-10T19:36:41Z

ls /etc/kubernetes/components/
more /etc/kubernetes/components/custom-resources.yaml

apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 172.31.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
    nodeAddressAutodetectionV4:
      canReach: 192.168.56.11

kubectl apply -f /opt/crd.yaml

kubectl get crds
NAME                                                  CREATED AT
samples.bookofkubernetes.com                          2025-03-10T19:56:36Z

kubectl apply -f /opt/somedata.yaml

kubectl get sam
NAME       AGE
somedata   14s

kubectl get samples
NAME       AGE
somedata   22s

kubectl get sample
NAME       AGE
somedata   24s

kubectl get sample somedata
NAME       AGE
somedata   29s

kubectl describe sample somedata
Name:         somedata
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  bookofkubernetes.com/v1
Kind:         Sample
Metadata:
  Creation Timestamp:  2025-03-10T20:05:31Z
  Generation:          1
  Resource Version:    7938
  UID:                 5c357800-64d9-4772-94de-ad846c677c7a
Spec:
  Value:  123
Events:   <none>

kubectl apply -f /opt/sample-reader.yaml
kubectl apply -f /opt/sa.yaml
kubectl apply -f /opt/watch.yaml

kubectl get pods

NAME                     READY   STATUS    RESTARTS   AGE
watch-6546479dd7-8djvh   1/1     Running   0          28s

kubectl logs watch-6546479dd7-8djvh
{
  "type": "ADDED",
  "object": {
    "apiVersion": "bookofkubernetes.com/v1",
    "kind": "Sample",
    "metadata": {
      "creationTimestamp": "2025-03-10T20:05:31Z",
      "name": "somedata",
      "namespace": "default",
      "resourceVersion": "7938",
      "uid": "5c357800-64d9-4772-94de-ad846c677c7a"
    },
    "spec": {
      "value": 123
    }
  },

kubectl apply -f /etc/kubernetes/components/postgres-operator.yaml

kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
postgres-operator-7dd5bc796c-f2tc8   1/1     Running   0          14s

kubectl get crd postgresqls.acid.zalan.do
NAME                        CREATED AT
postgresqls.acid.zalan.do   2025-03-10T20:17:34Z

kubectl apply -f /opt/pgsql.yaml

kubectl logs postgres-operator-7dd5bc796c-f2tc8

kubectl apply -k github.com/zalando/postgres-operator/manifests

kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
pgsql-cluster-0                     0/1     ContainerCreating   0          2m39s
postgres-operator-d85df4bbc-vff4j   1/1     Running             0          3m20s
watch-6546479dd7-8djvh              1/1     Running             0          31m

kubectl get sts
NAME            READY   AGE
pgsql-cluster   0/3     3m

kubectl logs pgsql-cluster-1
2025-03-10 21:10:27,893 INFO: Lock owner: pgsql-cluster-0; I am pgsql-cluster-1
2025-03-10 21:10:29,321 INFO: bootstrap from leader 'pgsql-cluster-0' in progress
2025-03-10 21:10:29,337 INFO: Lock owner: pgsql-cluster-0; I am pgsql-cluster-1
2025-03-10 21:10:29,337 INFO: establishing a new patroni heartbeat connection to postgres
2025-03-10 21:10:31,441 INFO: no action. I am (pgsql-cluster-1), a secondary, and following a leader (pgsql-cluster-0)

kubectl logs pgsql-cluster-0
2025-03-10 21:08:06,604 INFO: initialized a new cluster
2025-03-10 21:08:06,792 INFO: no action. I am (pgsql-cluster-0), the leader with the lock
2025-03-10 21:08:16,725 INFO: no action. I am (pgsql-cluster-0), the leader with the lock

kubectl logs pgsql-cluster-2
2025-03-10 21:11:19,527 INFO: Lock owner: pgsql-cluster-0; I am pgsql-cluster-2
2025-03-10 21:11:19,528 INFO: establishing a new patroni heartbeat connection to postgres
2025-03-10 21:11:22,534 INFO: no action. I am (pgsql-cluster-2), a secondary, and following a leader (pgsql-cluster-0)
2025-03-10 21:11:27,097 INFO: no action. I am (pgsql-cluster-2), a secondary, and following a leader (pgsql-cluster-0)

kubectl get svc
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP    16m
pgsql-cluster          ClusterIP   10.107.96.247    <none>        5432/TCP   5m25s
pgsql-cluster-config   ClusterIP   None             <none>        <none>     4m6s
pgsql-cluster-repl     ClusterIP   10.101.253.95    <none>        5432/TCP   5m25s
postgres-operator      ClusterIP   10.109.201.153   <none>        8080/TCP   6m37s

kubectl delete -f /opt/pgsql.yaml

kubectl logs postgres-operator-d85df4bbc-r2kmg

time="2025-03-10T21:13:07Z" level=debug msg="deleting pods" cluster-name=default/pgsql-cluster pkg=cluster worker=0
time="2025-03-10T21:13:07Z" level=debug msg="deleting pod \"default/pgsql-cluster-0\"" cluster-name=default/pgsql-cluster pkg=cluster worker=0
time="2025-03-10T21:13:07Z" level=debug msg="subscribing to pod \"default/pgsql-cluster-0\"" cluster-name=default/pgsql-cluster pkg=cluster worker=0