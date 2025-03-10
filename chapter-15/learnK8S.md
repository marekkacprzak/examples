kubectl get storageclass

NAME                 PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
longhorn (default)   driver.longhorn.io   Delete          Immediate           true                   25s

crictl ps --name 'longhorn.*|csi.*'

CONTAINER           IMAGE               CREATED             STATE               NAME                        ATTEMPT             POD ID
fbddd57739d2e       ec9b939801797       34 seconds ago      Running             csi-provisioner             1                   3d2af02152445
748622a9d2220       9f4c1b666bd8c       36 seconds ago      Running             longhorn-csi-plugin         0                   4df8deb386582
5b5afd756e490       494ea5379400e       39 seconds ago      Running             longhorn-liveness-probe     0                   4df8deb386582
248710eb2f034       d11bd1f0b3e57       41 seconds ago      Running             csi-resizer                 1                   fbecfd2fc01ff
d5a5dd1c8cc79       118137d698bbd       50 seconds ago      Running             csi-snapshotter             0                   94bdbdcd704f6
978b7663fb923       6bc8bdd63e408       58 seconds ago      Running             csi-attacher                0                   f24c591931451
1425e4f85c2e6       73ddb59b21918       2 minutes ago       Running             csi-node-driver-registrar   0                   6d2180d24f855
a5d0b7fab2fdc       9f4c1b666bd8c       2 minutes ago       Running             longhorn-manager            1                   d251218c43001
4aada9fd2bbf6       b2c0fe47b0708       3 minutes ago       Running             calico-csi                  0                   6d2180d24f855

CID=$(crictl ps -q --name csi-attacher)
crictl inspect $CID

 "mounts": [
      {
        "containerPath": "/csi/",
        "hostPath": "/var/lib/kubelet/plugins/driver.longhorn.io",
        "propagation": "PROPAGATION_PRIVATE",
        "readonly": false,
        "selinuxRelabel": false
      },
 "envs": [
        {
          "key": "ADDRESS",
          "value": "/csi/csi.sock"
        },

ls -l /var/lib/kubelet/plugins/driver.longhorn.io

total 0
srwxr-xr-x 1 root root 0 Mar 10 16:07 csi.sock

kubectl apply -f /opt/pgsql-set.yaml

kubectl get pods

kubectl get pods -o wide
NAME         READY   STATUS    RESTARTS   AGE     IP              NODE     NOMINATED NODE   READINESS GATES
postgres-0   1/1     Running   0          3m29s   172.31.25.212   host03   <none>           <none>
postgres-1   1/1     Running   0          88s     172.31.89.203   host02   <none>           <none>

kubectl exec -it postgres-0 -- /bin/sh

findmnt /data

TARGET SOURCE                                                 FSTYPE OPTIONS
/data  /dev/longhorn/pvc-7ca849f2-6169-4bdb-8877-7bc0b9633e44 ext4   rw,relatime

kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   21m
postgres     ClusterIP   None         <none>        <none>    4m57s

kubectl run -it --image=alpine --restart=Never alpine

ping -c 1 postgres-0.postgres.default.svc
PING postgres-0.postgres.default.svc (172.31.25.212): 56 data bytes
64 bytes from 172.31.25.212: seq=0 ttl=63 time=0.288 ms

--- postgres-0.postgres.default.svc ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.288/0.288/0.288 ms

ping -c 1 postgres-1.postgres.default.svc
PING postgres-1.postgres.default.svc (172.31.89.203): 56 data bytes
64 bytes from 172.31.89.203: seq=0 ttl=62 time=13.162 ms

--- postgres-1.postgres.default.svc ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 13.162/13.162/13.162 ms

exit

kubectl get pvc
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-volume-postgres-0   Bound    pvc-7ca849f2-6169-4bdb-8877-7bc0b9633e44   1Gi        RWO            longhorn       7m18s
postgres-volume-postgres-1   Bound    pvc-a46dbc93-3617-42c9-8fe6-c87b3e5ec059   1Gi        RWO            longhorn       5m16s

kubectl delete -f /opt/pgsql-set.yaml

kubectl get pvc
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-volume-postgres-0   Bound    pvc-7ca849f2-6169-4bdb-8877-7bc0b9633e44   1Gi        RWO            longhorn       7m57s
postgres-volume-postgres-1   Bound    pvc-a46dbc93-3617-42c9-8fe6-c87b3e5ec059   1Gi        RWO            longhorn       5m55s

kubectl apply -f /opt/pvc.yaml

kubectl get pvc
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-storage                Bound    pvc-bf874e2d-bfec-4169-b5fb-5ad4b81a0891   100Mi      RWO            longhorn       9s

kubectl apply -f /opt/pvc-man.yaml

kubectl get pvc
NAME                         STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
manual                       Pending                                                                        manual         8s
nginx-storage                Bound     pvc-bf874e2d-bfec-4169-b5fb-5ad4b81a0891   100Mi      RWO            longhorn       54s

kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                STORAGECLASS   REASON   AGE
pvc-7ca849f2-6169-4bdb-8877-7bc0b9633e44   1Gi        RWO            Delete           Bound    default/postgres-volume-postgres-0   longhorn                9m54s
pvc-a46dbc93-3617-42c9-8fe6-c87b3e5ec059   1Gi        RWO            Delete           Bound    default/postgres-volume-postgres-1   longhorn                7m49s
pvc-bf874e2d-bfec-4169-b5fb-5ad4b81a0891   100Mi      RWO            Delete           Bound    default/nginx-storage                longhorn                79s

kubectl apply -f /opt/pv.yaml

kubectl get pvc
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
manual                       Bound    manual                                     100Mi      RWO            manual         83s

kubectl apply -f /opt/nginx.yaml

IP=$(kubectl get po -l app=nginx -o jsonpath='{..podIP}')

curl -v http://$IP

*   Trying 172.31.25.214:80...
* TCP_NODELAY set
* Connected to 172.31.25.214 (172.31.25.214) port 80 (#0)
> GET / HTTP/1.1
> Host: 172.31.25.214
> User-Agent: curl/7.68.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 403 Forbidden
< Server: nginx/1.27.4
< Date: Mon, 10 Mar 2025 16:23:16 GMT
< Content-Type: text/html
< Content-Length: 153
< Connection: keep-alive
<
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.27.4</center>
</body>
</html>
* Connection #0 to host 172.31.25.214 left intact

POD=$(kubectl get po -l app=nginx -o jsonpath='{..metadata.name}')

kubectl cp /opt/index.html $POD:/usr/share/nginx/html

curl -v http://$IP

*   Trying 172.31.25.214:80...
* TCP_NODELAY set
* Connected to 172.31.25.214 (172.31.25.214) port 80 (#0)
> GET / HTTP/1.1
> Host: 172.31.25.214
> User-Agent: curl/7.68.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.27.4
< Date: Mon, 10 Mar 2025 16:25:22 GMT
< Content-Type: text/html
< Content-Length: 120
< Last-Modified: Mon, 10 Mar 2025 16:25:12 GMT
< Connection: keep-alive
< ETag: "67cf1268-78"
< Accept-Ranges: bytes
<
<html>
  <head>
    <title>Hello, World</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
  </body>
</html>
* Connection #0 to host 172.31.25.214 left intact

kubectl scale --replicas=3 deployment/nginx

kubectl get pods

NAME                     READY   STATUS              RESTARTS   AGE
alpine                   0/1     Completed           0          11m
nginx-55f8d45479-lfwgh   0/1     ContainerCreating   0          26s
nginx-55f8d45479-mg7f6   1/1     Running             0          5m29s
nginx-55f8d45479-pfsf8   0/1     ContainerCreating   0          26s

kubectl describe pod/nginx-55f8d45479-lfwgh

Name:             nginx-55f8d45479-lfwgh
Status:           Pending
Events:
  Type     Reason              Age   From                     Message
  ----     ------              ----  ----                     -------
  Warning  FailedAttachVolume  56s   attachdetach-controller  Multi-Attach error for volume "pvc-bf874e2d-bfec-4169-b5fb-5ad4b81a0891" Volume is already used by pod(s) nginx-55f8d45479-mg7f6

kubectl delete deploy/nginx pvc/nginx-storage

kubectl apply -f /opt/pvc-rwx.yaml

kubectl apply -f /opt/nginx.yaml

kubectl get pods

POD=$(kubectl get po -l app=nginx -o jsonpath='{..metadata.name}')

kubectl cp /opt/index.html $POD:/usr/share/nginx/html

IP=$(kubectl get po -l app=nginx -o jsonpath='{..podIP}')

curl -v http://$IP

*   Trying 172.31.25.216:80...
* TCP_NODELAY set
* Connected to 172.31.25.216 (172.31.25.216) port 80 (#0)
> GET / HTTP/1.1
> Host: 172.31.25.216
> User-Agent: curl/7.68.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.27.4
< Date: Mon, 10 Mar 2025 16:32:11 GMT
< Content-Type: text/html
< Content-Length: 120
< Last-Modified: Mon, 10 Mar 2025 16:31:46 GMT
< Connection: keep-alive
< ETag: "67cf13f2-78"
< Accept-Ranges: bytes
<
<html>
  <head>
    <title>Hello, World</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
  </body>
</html>
* Connection #0 to host 172.31.25.216 left intact
