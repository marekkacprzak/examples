kubectl get pods -o wide
kubectl apply -f /opt/nginx-deploy.yaml
get deployment nginx -o wide
get po -o wide
kubectl get replicasets
kubectl describe pod nginx-868878b758-6rwv9
kubectl describe deployment nginx
kubectl get all -l app=nginx
kubectl scale --replicas=4 deployment nginx
kubectl apply -f nginx-scaler.yaml
 kubectl get hpa
NAME    REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx   Deployment/nginx   0%/50%    1         10        3          36s
kubectl apply -f sleep-job.yaml
kubectl get jobs
NAME    COMPLETIONS   DURATION   AGE
sleep   1/1           38s        72s
kubectl logs nginx-868878b758-2ggmh

kubectl apply -f sleep-set.yaml
kubectl get statefulset
kubectl get po -o wide
sleep-0                  1/1     Running     0          3m3s    172.31.15.104   host04   <none>           <none>
sleep-1                  1/1     Running     0          2m20s   172.31.25.220   host03   <none>           <none>
exec sleep-0 -- /bin/sh -c 'hostname > /storagedir/myhost'
kubectl exec sleep-0 -- /bin/cat /storagedir/myhost
kubectl exec sleep-1 -- /bin/sh -c 'hostname > /storagedir/myhost'
kubectl delete pod sleep-0
kubectl exec sleep-0 -- /bin/cat /storagedir/myhost
sleep-0
kubectl -n calico-system get daemonset calico-node -o yaml
kubectl -n calico-system get pods -l k8s-app=calico-node -o wide
