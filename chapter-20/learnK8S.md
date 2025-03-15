kubectl delete namespace todo
kubectl apply -k github.com/Zalando/postgres-operator/manifests
kubectl create namespace todo
kubectl -n todo apply -f database.yaml
kubectl get secret -n todo
kubectl edit configmap postgres-operator
kubectl get sts -n todo

kubectl exec -it todo-db-0 -n todo -- cat /home/postgres/pgdata/pgroot/data/pg_hba.conf
kubectl exec -it todo-db-0 -n todo -- psql -U todo -d todo -c "SHOW ssl;"
kubectl get sts -n todo
kubectl get secret -n todo
kubectl get secret -n todo
kubectl get secret todo.todo-db.credentials.postgresql.acid.zalan.do -n todo -o yaml
kubectl edit secret todo.todo-db.credentials.postgresql.acid.zalan.do -n todo

echo "dG9kbw==" | base64 --decode
echo "b0g3a2o4Y0dieE9kMU45OERzWkhlQU13R1lNTDBBZEZPbXFBeHFDMVdOaktrR2dMNmxqWmRsdFZWZXRSaXpnZg==" | base64 --decode
kubectl exec -it todo-db-0 -n todo -- psql -U postgres -d postgres
CREATE ROLE todo WITH LOGIN PASSWORD 'oH7kj8cGbxOd1N98DsZHeAMwGYML0AdFOmqAxqC1WNjKkGgL6ljZdltVVetRizgf' SUPERUSER CREATE
DB;
GRANT ALL PRIVILEGES ON DATABASE todo TO todo;
CREATE DATABASE todo OWNER todo;

SELECT usename, passwd FROM pg_shadow WHERE usename = 'todo';
ALTER USER todo WITH PASSWORD 'NoweSuperHaslo';

kubectl -n todo apply -f application.yaml

kubectl -n todo apply -f service.yaml
kubectl -n todo apply -f scaler.yaml
kubectl -n todo get svc

curl http://10.109.145.98:5000

kubectl drain --ignore-daemonsets --delete-emptydir-data host05
kubectl delete  node host05

kubectl label pod todo-db-0 -n todo spilo-role=master --overwrite
kubectl label pod todo-db-1 -n todo spilo-role=replica --overwrite
kubectl label pod todo-db-2 -n todo spilo-role=replica --overwrite
kubectl rollout restart deployment postgres-operator

kubectl exec -it todo-db-0 -n todo -- psql -U postgres -c "\du"
kubectl get statefulsets -n todo

kubectl logs todo-758c7878c-hz85d -n todo --previous
kubectl top pod todo-758c7878c-hz85d -n todo

curl -v http://host01:5000/

sed -i 's/\r$//' /opt/api-metrics.sh
./api-metrics.sh
sed -i 's/\r$//' /opt/api-server-metrics.sh
/opt/api-server-metrics.sh
curl http://host01:5000/metrics

sed -i 's/\r$//' /opt/install-kube-prometheus.sh

/opt/install-kube-prometheus.sh

kubectl -n monitoring get pods

kubectl -n monitoring describe prometheus

kubectl -n monitoring get servicemonitor

NAME                      AGE
alertmanager-main         4m52s
blackbox-exporter         4m52s
coredns                   4m45s
grafana                   4m46s
kube-apiserver            4m45s
kube-controller-manager   4m45s
kube-scheduler            4m45s
kube-state-metrics        4m45s
kubelet                   4m45s
node-exporter             4m45s
prometheus-adapter        4m41s
prometheus-k8s            4m42s
prometheus-operator       4m40s

kubectl -n todo apply -f /opt/rbac.yaml

kubectl -n todo apply -f /opt/svc-mon.yaml

