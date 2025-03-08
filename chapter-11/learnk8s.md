curl https://192.168.56.11:6443/

curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl -kv https://192.168.56.11:6443/

* Server certificate:
*  subject: CN=kube-apiserver
*  start date: Mar  7 15:50:38 2025 GMT
*  expire date: Mar  7 15:55:38 2026 GMT
*  issuer: CN=Kubernetes

cp /etc/kubernetes/pki/ca.crt .
openssl x509 -in ca.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 6544295599679517974 (0x5ad2022cbc0cb516)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Mar  7 15:50:38 2025 GMT
            Not After : Mar  5 15:55:38 2035 GMT
        Subject: CN = kubernetes
        Subject Public Key Info:

curl --cacert ca.crt https://192.168.56.11:6443/
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {},
  "code": 403
}

kubeadm certs check-expiration

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Mar 07, 2026 15:55 UTC   364d            ca                      no
apiserver                  Mar 07, 2026 15:55 UTC   364d            ca                      no
apiserver-etcd-client      Mar 07, 2026 15:55 UTC   364d            etcd-ca                 no
apiserver-kubelet-client   Mar 07, 2026 15:55 UTC   364d            ca                      no
controller-manager.conf    Mar 07, 2026 15:55 UTC   364d            ca                      no
etcd-healthcheck-client    Mar 07, 2026 15:55 UTC   364d            etcd-ca                 no
etcd-peer                  Mar 07, 2026 15:55 UTC   364d            etcd-ca                 no
etcd-server                Mar 07, 2026 15:55 UTC   364d            etcd-ca                 no
front-proxy-client         Mar 07, 2026 15:55 UTC   364d            front-proxy-ca          no
scheduler.conf             Mar 07, 2026 15:55 UTC   364d            ca                      no

more /etc/kubernetes/admin.conf

echo $KUBECONFIG
/etc/kubernetes/admin.conf

kubeadm kubeconfig user --client-name=me --config /etc/kubernetes/kubeadm-init.yaml > kubeconfig

cat kubeconfig

KUBECONFIG=kubeconfig kubectl get pods
Error from server (Forbidden): pods is forbidden: User "me" cannot list resource "pods" in API group "" in the namespace "default"

TOKEN=$(kubeadm token create)
kubectl -n kube-system get secret

NAME                     TYPE                            DATA   AGE
bootstrap-token-iza9lf   bootstrap.kubernetes.io/token   6      23s
kubeadm-certs            Opaque                          8      22m

curl --cacert ca.crt -H "Authorization: Bearer $TOKEN" https://192.168.56.11:6443/apis/certificates.k8s.io/v1/certificatesigningrequests

{
  "kind": "CertificateSigningRequestList",
  "apiVersion": "certificates.k8s.io/v1",
  "metadata": {
    "resourceVersion": "7441"
  },
  "items": [

kubectl create namespace sample

kubectl apply -f /opt/read-pods-sa.yaml

kubectl -n sample get serviceaccounts
NAME        SECRETS   AGE
default     0         62s
read-pods   0         28s

kubectl apply -f /opt/pod-reader.yaml
kubectl apply -f /opt/read-pods-bind.yaml
kubectl apply -f /opt/read-pods-deploy.yaml

kubectl -n sample get pods
NAME                         READY   STATUS    RESTARTS   AGE
read-pods-58db85d957-zjb2j   1/1     Running   0          4m31s

kubectl -n sample exec -it read-pods-58db85d957-zjb2j -- /bin/sh

cd /run/secrets/kubernetes.io/serviceaccount/

ls -l
total 0
lrwxrwxrwx    1 root     root            13 Mar  7 16:39 ca.crt -> ..data/ca.crt
lrwxrwxrwx    1 root     root            16 Mar  7 16:39 namespace -> ..data/namespace
lrwxrwxrwx    1 root     root            12 Mar  7 16:39 token -> ..data/token

TOKEN=$(cat token)

apk add curl

curl --cacert ca.crt -H "Authorization: Bearer $TOKEN" https://kubernetes.default.svc/api/v1/namespaces/sample/pods

{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "4003"
  },
  "items": [

curl --cacert ca.crt -H "Authorization: Bearer $TOKEN" https://kubernetes.default.svc/api/v1/namespaces/kube-system/pods
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:sample:read-pods\" cannot list resource \"pods\" in API group \"\" in the namespace \"kube-system\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}

kubectl get clusterrole edit -o yaml

aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.authorization.k8s.io/aggregate-to-edit: "true"
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole

kubectl apply -f /opt/edit-bind.yaml

KUBECONFIG=kubeconfig kubectl get pods
Error from server (Forbidden): pods is forbidden: User "me" cannot list resource "pods" in API group "" in the namespace "default"

KUBECONFIG=kubeconfig kubectl -n sample get pods
NAME                         READY   STATUS    RESTARTS   AGE
read-pods-58db85d957-zjb2j   1/1     Running   0          11m

