kubectl apply -f /opt/nginx-exec.yaml
kubectl get pods
kubectl describe deployment nginx
kubectl patch deploy nginx --patch-file /opt/nginx-404.yaml
kubectl get pods
kubectl describe pod
kubectl get pods
NAME                     READY   STATUS    RESTARTS      AGE
nginx-6699fcbf58-wnhxz   1/1     Running   3 (11s ago)   114s
kubectl apply -f /opt/nginx-http.yaml
kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-584bd96667-9qt84   1/1     Running   0          49s
10.0.2.15 - - [09/Mar/2025:19:45:43 +0000] "GET / HTTP/1.1" 200 615 "-" "kube-probe/1.28" "-"
10.0.2.15 - - [09/Mar/2025:19:45:53 +0000] "GET / HTTP/1.1" 200 615 "-" "kube-probe/1.28" "-"
10.0.2.15 - - [09/Mar/2025:19:46:03 +0000] "GET / HTTP/1.1" 200 615 "-" "kube-probe/1.28" "-"
kubectl delete deployment nginx
kubectl apply -f /opt/postgres-tcp.yaml
kubectl get pods
kubectl describe pod postgres-688c58b85b-6wp8n
Events:
  Type     Reason     Age                    From               Message
  ----     ------     ----                   ----               -------
  Normal   Scheduled  9m4s                   default-scheduler  Successfully assigned default/postgres-688c58b85b-6wp8n to host03
  Normal   Pulled     7m54s                  kubelet            Successfully pulled image "postgres" in 51.989s (51.989s including waiting)
  Warning  Unhealthy  7m28s (x3 over 7m48s)  kubelet            Liveness probe failed: dial tcp 172.31.25.215:5432: i/o timeout
  Normal   Killing    7m28s                  kubelet            Container postgres failed liveness probe, will be restarted
  Normal   Pulling    6m55s (x2 over 8m46s)  kubelet            Pulling image "postgres"
  Normal   Created    6m53s (x2 over 7m53s)  kubelet            Created container postgres
  Normal   Started    6m53s (x2 over 7m51s)  kubelet            Started container postgres
  Normal   Pulled     6m53s                  kubelet            Successfully pulled image "postgres" in 1.488s (1.488s including waiting)
  Warning  Unhealthy  6m49s                  kubelet            Liveness probe failed: dial tcp 172.31.25.215:5432: connect: connection refused
kubectl delete deploy postgres
kubectl apply -f /opt/nginx-ready.yaml
kubectl get pods
kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
nginx        ClusterIP   10.111.164.9   <none>        80/TCP    40s
curl http://10.111.164.9
curl: (7) Failed to connect to 10.111.164.9 port 80: Connection refused
kubectl describe pod
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  103s               default-scheduler  Successfully assigned default/nginx-78998fdb48-4tlhj to host01
  Normal   Pulling    99s                kubelet            Pulling image "nginx"
  Normal   Pulled     87s                kubelet            Successfully pulled image "nginx" in 12.628s (12.63s including waiting)
  Normal   Created    86s                kubelet            Created container nginx
  Normal   Started    86s                kubelet            Started container nginx
  Warning  Unhealthy  2s (x13 over 85s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 404

iptables-save | grep default/nginx

-A KUBE-SERVICES -d 10.111.164.9/32 -p tcp -m comment --comment "default/nginx has no endpoints" -m tcp --dport 80 -j REJECT --reject-with icmp-port-unreachable

kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-78998fdb48-4tlhj   0/1     Running   0          3m28s
nginx-78998fdb48-cxtlg   0/1     Running   0          3m28s
nginx-78998fdb48-h776l   0/1     Running   0          3m28s

kubectl exec -it nginx-78998fdb48-4tlhj -- cp -v /etc/hostname /usr/share/nginx/html/ready

curl http://10.111.164.9/ready
nginx-78998fdb48-4tlhj

kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-78998fdb48-4tlhj   1/1     Running   0          5m47s
nginx-78998fdb48-cxtlg   0/1     Running   0          5m47s
nginx-78998fdb48-h776l   0/1     Running   0          5m47s

kubectl exec -it nginx-78998fdb48-cxtlg -- cp -v /etc/hostname /usr/share/nginx/html/ready
kubectl exec -it nginx-78998fdb48-h776l -- cp -v /etc/hostname /usr/share/nginx/html/ready

curl http://10.111.164.9/ready
nginx-78998fdb48-h776l
curl http://10.111.164.9/ready
nginx-78998fdb48-4tlhj
kubectl describe service nginx
Endpoints:         172.31.239.203:80,172.31.25.216:80,172.31.89.202:80

for i in $(seq 1 5); do curl http://10.111.164.9/ready; done

nginx-78998fdb48-4tlhj
nginx-78998fdb48-cxtlg
nginx-78998fdb48-h776l
nginx-78998fdb48-cxtlg
nginx-78998fdb48-4tlhj

