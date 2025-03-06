kubectl apply -f /opt/nginx-selector.yaml
kubectl get pods -o wide
kubectl describe pod nginx

Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  104s  default-scheduler  0/3 nodes are available: 3 node(s) didn't match Pod's node affinity/selector. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling..

kubectl get nodes

NAME     STATUS   ROLES           AGE     VERSION
host01   Ready    control-plane   10m     v1.28.0
host02   Ready    control-plane   8m52s   v1.28.0
host03   Ready    control-plane   7m7s    v1.28.0

kubectl label nodes host02 purpose=special
 kubectl get pods -o wide
NAME    READY   STATUS              RESTARTS   AGE     IP       NODE     NOMINATED NODE   READINESS GATES
nginx   0/1     ContainerCreating   0          3m18s   <none>   host02   <none>           <none>

kubectl describe pod nginx

Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Warning  FailedScheduling  3m54s  default-scheduler  0/3 nodes are available: 3 node(s) didn't match Pod's node affinity/selector. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling..
  Normal   Scheduled         44s    default-scheduler  Successfully assigned default/nginx to host02
  Normal   Pulling           35s    kubelet            Pulling image "nginx"
  Normal   Pulled            15s    kubelet            Successfully pulled image "nginx" in 19.9s (19.9s including waiting)
  Normal   Created           15s    kubelet            Created container nginx
  Normal   Started           14s    kubelet            Started container nginx

kubectl delete -f /opt/nginx-selector.yaml
unlabel:
kubectl label nodes host02 purpose-

kubectl apply -f /opt/sleep-multiple.yaml

kubectl get pods -o wide

kubectl describe pod sleep

Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  44s   default-scheduler  0/3 nodes are available: 3 Insufficient cpu. preemption: 0/3 nodes are available: 3 No preemption victims found for incoming pod..

kubectl top node
NAME     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
host01   1823m        45%    1394Mi          74%
host02   2124m        53%    1340Mi          71%
host03   2223m        55%    1387Mi          74%

kubectl apply -f /opt/sleep-sensible.yaml
The Pod "sleep" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`,`spec.initContainers[*].image`,`spec.activeDeadlineSeconds`,`spec.tolerations` (only additions to existing tolerations),`spec.terminationGracePeriodSeconds` (allow it to be set to 1 if it was previously negative)

kubectl apply -f /opt/sleep-sensible.yaml

kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
sleep   2/2     Running   0          33s   172.31.239.203   host01   <none>           <none>

kubectl apply -f /opt/nginx-typo.yaml

kubectl get pods -o wide
NAME    READY   STATUS         RESTARTS   AGE   IP              NODE     NOMINATED NODE   READINESS GATES
nginx   0/1     ErrImagePull   0          38s   172.31.89.203   host02   <none>           <none>

kubectl describe pod nginx

Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  60s                default-scheduler  Successfully assigned default/nginx to host02
  Normal   BackOff    24s (x2 over 54s)  kubelet            Back-off pulling image "nginz"
  Warning  Failed     24s (x2 over 54s)  kubelet            Error: ImagePullBackOff
  Normal   Pulling    9s (x3 over 56s)   kubelet            Pulling image "nginz"
  Warning  Failed     7s (x3 over 54s)   kubelet            Failed to pull image "nginz": failed to pull and unpack image "docker.io/library/nginz:latest": failed to resolve reference "docker.io/library/nginz:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     7s (x3 over 54s)   kubelet            Error: ErrImagePull

kubectl set image pod nginx nginx=nginx # update pod nginx , container name nginx with image name nginx
pod/nginx image updated

kubectl apply -f /opt/postgres-misconfig.yaml

kubectl get pods
NAME       READY   STATUS              RESTARTS   AGE
postgres   0/1     ContainerCreating   0          15s

kubectl describe pod postgres

Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  75s                default-scheduler  Successfully assigned default/postgres to host02
  Normal   Pulled     31s                kubelet            Successfully pulled image "postgres" in 39.202s (39.202s including waiting)
  Normal   Pulled     25s                kubelet            Successfully pulled image "postgres" in 1.692s (1.692s including waiting)
  Normal   Pulling    11s (x3 over 71s)  kubelet            Pulling image "postgres"
  Normal   Created    9s (x3 over 31s)   kubelet            Created container postgres
  Normal   Pulled     9s                 kubelet            Successfully pulled image "postgres" in 1.769s (1.769s including waiting)
  Normal   Started    8s (x3 over 29s)   kubelet            Started container postgres
  Warning  BackOff    6s (x2 over 22s)   kubelet            Back-off restarting failed container postgres in pod postgres_default(c0c3e124-fc6e-4e6c-9829-c125e34a7881)

kubectl get pods
NAME       READY   STATUS             RESTARTS      AGE
postgres   0/1     CrashLoopBackOff   3 (25s ago)   2m3s

kubectl logs postgres
Error: Database is uninitialized and superuser password is not specified.
       You must specify POSTGRES_PASSWORD to a non-empty value for the
       superuser. For example, "-e POSTGRES_PASSWORD=password" on "docker run".

       You may also use "POSTGRES_HOST_AUTH_METHOD=trust" to allow all
       connections without a password. This is *not* recommended.

       See PostgreSQL documentation about "trust":
       https://www.postgresql.org/docs/current/auth-trust.html

kubectl delete pod postgres

kubectl apply -f /opt/postgres-fixed.yaml

kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
postgres   1/1     Running   0          17s

kubectl delete -f /opt/postgres-fixed.yaml

https://docs.docker.com/engine/install/ubuntu/

kubectl apply -f /opt/crasher-deploy.yaml

kubectl get pod
NAME                       READY   STATUS   RESTARTS     AGE
crasher-5db57665ff-tx4xm   0/1     Error    1 (4s ago)   17s

kubectl get pod crasher-5db57665ff-tx4xm -o json | jq '.status.containerStatuses[].lastState.terminated.exitCode'
139
kubectl logs crasher-5db57665ff-tx4xm

kubectl edit deployment crasher

 args: ["/bin/sleep","infinity"]

kubectl get pod
NAME                       READY   STATUS    RESTARTS   AGE
crasher-7cc6cd8fc6-4fkrf   1/1     Running   0          27s

kubectl exec -it crasher-7cc6cd8fc6-4fkrf -- /bin/sh

/crasher
Segmentation fault (core dumped)

apk add gdb

gdbserver localhost:2345 /crasher

kubectl port-forward pods/crasher-7cc6cd8fc6-4fkrf 2345:2345

cd /opt/
gdb -q
target remote localhost:2345
0x00007ffff7fbfbde in _dlstart () from target:/lib/ld-musl-x86_64.so.1
continue
Program received signal SIGSEGV, Segmentation fault.
main () at crasher.c:3
warning: Source file is more recent than executable.
3         s[2] = '3';



