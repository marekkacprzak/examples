systemctl status kubelet

Mar 07 19:04:16 host01 kubelet[15465]: E0307 19:04:16.239779   15465 kuberuntime_manager.go:1119] "CreatePodSandbox for>
Mar 07 19:04:16 host01 kubelet[15465]: E0307 19:04:16.239868   15465 pod_workers.go:1300] "Error syncing pod, skipping">
Mar 07 19:04:16 host01 kubelet[15465]: E0307 19:04:16.325206   15465 remote_runtime.go:193] "RunPodSandbox from runtime>
Mar 07 19:04:16 host01 kubelet[15465]: E0307 19:04:16.325336   15465 kuberuntime_sandbox.go:72] "Failed to create sandb>
Mar 07 19:04:16 host01 kubelet[15465]: E0307 19:04:16.325395   15465 kuberuntime_manager.go:1119] "CreatePodSandbox for>
Mar 07 19:04:16 host01 kubelet[15465]: E0307 19:04:16.328802   15465 pod_workers.go:1300] "Error syncing pod, skipping">
Mar 07 19:04:16 host01 kubelet[15465]: I0307 19:04:16.851943   15465 pod_container_deletor.go:80] "Container not found >
Mar 07 19:04:17 host01 kubelet[15465]: I0307 19:04:17.021229   15465 pod_container_deletor.go:80] "Container not found >
Mar 07 19:04:19 host01 kubelet[15465]: I0307 19:04:19.329145   15465 pod_startup_latency_tracker.go:102] "Observed pod >
Mar 07 19:04:54 host01 kubelet[15465]: I0307 19:04:54.886859   15465 pod_startup_latency_tracker.go:102] "Observed pod >

strings /proc/$(pgrep kubelet)/cmdline

/usr/bin/kubelet
--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf
--kubeconfig=/etc/kubernetes/kubelet.conf
--config=/var/lib/kubelet/config.yaml
--container-runtime-endpoint=unix:///var/run/containerd/containerd.sock
--node-ip=192.168.56.11
--pod-infra-container-image=registry.k8s.io/pause:3.9

more /var/lib/kubelet/config.yaml

authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt

crictl images

kubectl apply -f /opt/pod.yaml

kubectl exec -it debug -- /bin/sh

mount | grep resolv
/dev/sda1 on /etc/resolv.conf type ext4 (rw,relatime)

cat /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5

mount | grep run
tmpfs on /run/secrets/kubernetes.io/serviceaccount type tmpfs (ro,relatime,size=3861992k)

more /var/lib/kubelet/config.yaml

staticPodPath: /etc/kubernetes/manifests

ls -l /etc/kubernetes/manifests/
total 16
-rw------- 1 root root 2399 Mar  7 18:59 etcd.yaml
-rw------- 1 root root 4078 Mar  7 18:59 kube-apiserver.yaml
-rw------- 1 root root 3543 Mar  7 18:59 kube-controller-manager.yaml
-rw------- 1 root root 1463 Mar  7 18:59 kube-scheduler.yaml

kubectl apply -f /opt/deploy.yaml
deployment.apps/debug created

kubectl get pod -o wide

kubectl drain --ignore-daemonsets host04

kubectl get nodes
NAME     STATUS                     ROLES           AGE    VERSION
host01   Ready                      control-plane   149m   v1.28.0
host02   Ready                      control-plane   148m   v1.28.0
host03   Ready                      control-plane   146m   v1.28.0
host04   Ready,SchedulingDisabled   <none>          146m   v1.28.0

kubectl delete node host04

root@host04:/home/vagrant# systemctl restart kubelet

kubectl scale deployment debug --replicas=1

kubectl top nodes
NAME     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
host01   802m         20%    1413Mi          75%
host02   791m         19%    1291Mi          69%
host03   1250m        31%    1389Mi          74%
host04   432m         10%    860Mi           45%

root@host04:/home/vagrant# cat /opt/node-evict.yaml >> /var/lib/kubelet/config.yaml

kubectl describe node host04

 Normal   NodeHasInsufficientMemory  88s                  kubelet          Node host04 status is now: NodeHasInsufficientMemory
  Warning  EvictionThresholdMet       19s (x6 over 97s)    kubelet          Attempting to reclaim memory

root@host01:/home/vagrant# kubectl get pod
NAME                     READY   STATUS                   RESTARTS   AGE
debug-6f788595bf-4x79s   1/1     Running                  0          119s
debug-6f788595bf-8m6rm   1/1     Running                  0          7m10s
debug-6f788595bf-9xj6m   1/1     Running                  0          4m17s
debug-6f788595bf-dgdzf   1/1     Running                  0          2m32s
debug-6f788595bf-fskd7   1/1     Running                  0          4m17s
debug-6f788595bf-gq4dg   1/1     Running                  0          2m32s
debug-6f788595bf-hdmzw   1/1     Running                  0          4m17s
debug-6f788595bf-lfjk9   1/1     Running                  0          4m17s
debug-6f788595bf-mmgmh   1/1     Running                  0          4m17s
debug-6f788595bf-ng8xh   0/1     OutOfmemory              0          4m17s
debug-6f788595bf-p25sl   1/1     Running                  0          4m17s
debug-6f788595bf-p97lm   0/1     OutOfmemory              0          4m17s
debug-6f788595bf-sz2wb   0/1     ContainerStatusUnknown   1          4m17s
debug-6f788595bf-tx8xn   0/1     OutOfmemory              0          4m17s

sed -i '/^evictionHard/,+2d' /var/lib/kubelet/config.yaml

kubectl scale deployment debug --replicas=1
kubectl scale deployment debug --replicas=12

host04:
iptables -I INPUT -s 192.168.56.11 -j DROP
iptables -I OUTPUT -d 192.168.56.11 -j DROP

kubectl get nodes
NAME     STATUS     ROLES           AGE     VERSION
host01   Ready      control-plane   4h22m   v1.28.0
host02   Ready      control-plane   4h21m   v1.28.0
host03   Ready      control-plane   4h19m   v1.28.0
host04   NotReady   <none>          161m    v1.28.0




