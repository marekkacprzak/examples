kubectl apply -f /opt/nginx-deploy.yaml
kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
nginx-7854ff8877-2sgsh   1/1     Running   0          35s   172.31.239.203   host01   <none>           <none>
nginx-7854ff8877-fs9hd   1/1     Running   0          35s   172.31.25.212    host03   <none>           <none>
nginx-7854ff8877-pkvxn   1/1     Running   0          35s   172.31.89.204    host02   <none>           <none>
nginx-7854ff8877-sn24d   1/1     Running   0          35s   172.31.89.203    host02   <none>           <none>
nginx-7854ff8877-w9bth   1/1     Running   0          35s   172.31.239.202   host01   <none>           <none>

curl -v http://172.31.239.203
kubectl apply -f /opt/nginx-service.yaml
kubectl get services
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   77m
nginx        ClusterIP   10.109.222.235   <none>        80/TCP    18s

curl -v http://10.109.222.235

kubectl describe service nginx
Name:              nginx
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.109.222.235
IPs:               10.109.222.235
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         172.31.239.202:80,172.31.239.203:80,172.31.25.212:80 + 2 more...
Session Affinity:  None
Events:            <none>

kubectl apply -f /opt/pod.yaml
kubectl exec -it pod -- wget -O - http://nginx
Connecting to nginx (10.109.222.235:80)
writing to stdout
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>

kubectl exec -it pod -- cat /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5

kubectl -n kube-system get services
NAME             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
kube-dns         ClusterIP   10.96.0.10    <none>        53/UDP,53/TCP,9153/TCP   84m
metrics-server   ClusterIP   10.99.0.214   <none>        443/TCP                  80m

kubectl exec -it pod -- wget -O - http://nginx.default.svc

kubectl exec -it pod -- apk add bind-tools

kubectl exec -it pod -- dig +search metrics-server

;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 55502
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1
;; AUTHORITY SECTION:

nie udalo sie odszukac bo pod jest w namespace default a metric w kube-system

kubectl exec -it pod -- dig +search metrics-server.kube-system
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
metrics-server.kube-system.svc.cluster.local. 30 IN A 10.99.0.214

kubectl exec -it pod -- ping -c 3 nginx
PING nginx (10.109.222.235): 56 data bytes
--- nginx ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss

ping nie dziala bo service ma tylko 80 port

iptables-save | grep 'default/nginx cluster IP'
-A KUBE-SERVICES -d 10.109.222.235/32 -p tcp -m comment --comment "default/nginx cluster IP" -m tcp --dport 80 -j KUBE-SVC-2CMXP7HKUVJN7L6M
-A KUBE-SVC-2CMXP7HKUVJN7L6M ! -s 172.31.0.0/16 -d 10.109.222.235/32 -p tcp -m comment --comment "default/nginx cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ

iptables-save | grep 'KUBE-SVC-2CMXP7HKUVJN7L6M'
-A KUBE-SVC-2CMXP7HKUVJN7L6M -m comment --comment "default/nginx -> 172.31.239.202:80" -m statistic --mode random --probability 0.20000000019 -j KUBE-SEP-SWVHRLELZHTPPAK4
-A KUBE-SVC-2CMXP7HKUVJN7L6M -m comment --comment "default/nginx -> 172.31.239.203:80" -m statistic --mode random --probability 0.25000000000 -j KUBE-SEP-P3HJIEJK53LVVBHR
-A KUBE-SVC-2CMXP7HKUVJN7L6M -m comment --comment "default/nginx -> 172.31.25.212:80" -m statistic --mode random --probability 0.33333333349 -j KUBE-SEP-YM26SYTQ5RTCYNFA
-A KUBE-SVC-2CMXP7HKUVJN7L6M -m comment --comment "default/nginx -> 172.31.89.203:80" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-GUZMNJ6JWEBQTSV6
-A KUBE-SVC-2CMXP7HKUVJN7L6M -m comment --comment "default/nginx -> 172.31.89.204:80" -j KUBE-SEP-J5BXKP62JQAGUC7L

iptables-save | grep 'KUBE-SEP-J5BXKP62JQAGUC7L'
-A KUBE-SEP-J5BXKP62JQAGUC7L -s 172.31.89.204/32 -m comment --comment "default/nginx" -j KUBE-MARK-MASQ
-A KUBE-SEP-J5BXKP62JQAGUC7L -p tcp -m comment --comment "default/nginx" -m tcp -j DNAT --to-destination 172.31.89.204:80
iptables-save | grep 'KUBE-SEP-SWVHRLELZHTPPAK4'
-A KUBE-SEP-SWVHRLELZHTPPAK4 -s 172.31.239.202/32 -m comment --comment "default/nginx" -j KUBE-MARK-MASQ
-A KUBE-SEP-SWVHRLELZHTPPAK4 -p tcp -m comment --comment "default/nginx" -m tcp -j DNAT --to-destination 172.31.239.202:80

kubectl patch svc nginx --patch-file /opt/nginx-nodeport.yaml
kubectl get service nginx
NAME    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx   NodePort   10.109.222.235   <none>        80:19442/TCP   93m

kubectl exec -it pod -- wget -O - http://nginx
wget -O - http://host01:19442

kubectl -n ingress-nginx get deploy
kubectl -n ingress-nginx get svc
NAME                                 TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                 AGE
ingress-nginx-controller             LoadBalancer   10.103.41.25   <pending>     80:80/TCP,443:443/TCP   179m
ingress-nginx-controller-admission   ClusterIP      10.97.39.37    <none>        443/TCP                 179m

curl -vH "Host:web01" http://192.168.56.11
curl -vH "Host:web01" http://host01


Since VirtualBox doesn’t have a cloud LoadBalancer, you can use MetalLB, a software-based LoadBalancer for on-prem environments.
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.9/config/manifests/metallb-native.yaml
kubectl get pods -n metallb-system

cat <<EOF | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.56.11/32  # Use your VirtualBox Host IP
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: first-adv
  namespace: metallb-system
EOF

kubectl delete -f /etc/kubernetes/components/ingress-controller.yaml
kubectl delete namespace ingress-nginx
kubectl apply -f /etc/kubernetes/components/ingress-controller.yaml
kubectl apply -f /etc/kubernetes/components/ingress-patch.yaml
kubectl get svc -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.108.207.47   192.168.56.11   80:32615/TCP,443:19910/TCP   7m35s
ingress-nginx-controller-admission   ClusterIP      10.97.135.194   <none>          443/TCP                      7m34s
http://web01/ now works

