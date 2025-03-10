kubectl apply -f /opt/pgsql-ext-cfg.yaml

kubectl get pods
NAME                        READY   STATUS                       RESTARTS   AGE
postgres-5484f8f4dd-2czzm   0/1     CreateContainerConfigError   0          67s

kubectl apply -f /opt/pgsql-cm.yaml

kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
postgres-5484f8f4dd-2czzm   1/1     Running   0          111s

kubectl exec -it postgres-5484f8f4dd-2czzm -- /bin/sh -c env

KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=postgres-5484f8f4dd-2czzm
HOME=/root
PG_VERSION=17.4-1.pgdg120+2
TERM=xterm
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
POSTGRES_PASSWORD=supersecret
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/postgresql/17/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
LANG=en_US.utf8
PG_MAJOR=17
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
GOSU_VERSION=1.17
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
PGDATA=/var/lib/postgresql/data

kubectl apply -f /opt/pgsql-secret.yaml

kubectl get secret pgsql -o json | jq .data
{
  "POSTGRES_PASSWORD": "c3VwZXJzZWNyZXQ="
}

kubectl apply -f /opt/pgsql-ext-sec.yaml

kubectl get pods
NAME                        READY   STATUS        RESTARTS   AGE
postgres-5484f8f4dd-2czzm   1/1     Terminating   0          4m22s
postgres-7d8c4877bb-qwtt2   1/1     Running       0          8s

kubectl exec -it postgres-7d8c4877bb-qwtt2 -- /bin/sh -c env

POSTGRES_PASSWORD=supersecret

kubectl delete deploy postgres

kubectl apply -f /opt/nginx-cm.yaml

kubectl apply -f /opt/nginx-deploy.yaml

IP=$(kubectl get po -l app=nginx -o jsonpath='{..podIP}')

curl http://$IP

<html>
  <head>
    <title>Hello World</title>
  </head>
  <body>
    <h1>Hello, World from a ConfigMap!</h1>
  </body>
</html>

kubectl get pods

NAME                     READY   STATUS    RESTARTS   AGE
nginx-656d658bd5-b2g65   1/1     Running   0          108s

kubectl exec -it nginx-656d658bd5-b2g65 -- /bin/mount

/dev/sda1 on /usr/share/nginx/html type ext4 (ro,relatime)

kubectl delete deploy nginx

source /opt/etcd-env

export ETCDCTL_API=3
export ETCDCTL_CERT=/etc/kubernetes/pki/apiserver-etcd-client.crt
export ETCDCTL_KEY=/etc/kubernetes/pki/apiserver-etcd-client.key
export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
export ETCDCTL_ENDPOINTS=https://192.168.56.13:2379

etcdctl member list
4461f2484ec5010b, started, host02, https://192.168.56.12:2380, https://192.168.56.12:2379
5a052b8cf4ed2fa2, started, host03, https://192.168.56.13:2380, https://192.168.56.13:2379
bf624c9e82dced96, started, host01, https://192.168.56.11:2380, https://192.168.56.11:2379

etcdctl get / --prefix --keys-only | grep configmaps

/registry/configmaps/default/nginx
/registry/configmaps/default/pgsql

etcdctl get / --prefix --keys-only | grep pgsql
/registry/configmaps/default/pgsql
/registry/secrets/default/pgsql

etcdctl -w json get /registry/secrets/default/pgsql | jq
{
  "header": {
    "cluster_id": 176150847236950400,
    "member_id": 6486638722701668000,
    "revision": 8343,
    "raft_term": 11
  },
  "kvs": [
    {
      "key": "L3JlZ2lzdHJ5L3NlY3JldHMvZGVmYXVsdC9wZ3NxbA==",
      "create_revision": 4062,
      "mod_revision": 4062,
      "version": 1,
      "value": "azhzAAoMCgJ2MRIGU2VjcmV0EqQECvcDCgVwZ3NxbBIAGgdkZWZhdWx0IgAqJGU3NjllZjQ3LTA3NDQtNDdiMi05MDI1LTNkNWM1OWE1YjBjZjIAOABCCAijury+BhAAYs0BCjBrdWJlY3RsLmt1YmVybmV0ZXMuaW8vbGFzdC1hcHBsaWVkLWNvbmZpZ3VyYXRpb24SmAF7ImFwaVZlcnNpb24iOiJ2MSIsImtpbmQiOiJTZWNyZXQiLCJtZXRhZGF0YSI6eyJhbm5vdGF0aW9ucyI6e30sIm5hbWUiOiJwZ3NxbCIsIm5hbWVzcGFjZSI6ImRlZmF1bHQifSwic3RyaW5nRGF0YSI6eyJQT1NUR1JFU19QQVNTV09SRCI6InN1cGVyc2VjcmV0In19CooB2wEKGWt1YmVjdGwtY2xpZW50LXNpZGUtYXBwbHkSBlVwZGF0ZRoCdjEiCAijury+BhAAMghGaWVsZHNWMTqbAQqYAXsiZjpkYXRhIjp7Ii4iOnt9LCJmOlBPU1RHUkVTX1BBU1NXT1JEIjp7fX0sImY6bWV0YWRhdGEiOnsiZjphbm5vdGF0aW9ucyI6eyIuIjp7fSwiZjprdWJlY3RsLmt1YmVybmV0ZXMuaW8vbGFzdC1hcHBsaWVkLWNvbmZpZ3VyYXRpb24iOnt9fX0sImY6dHlwZSI6e319QgASIAoRUE9TVEdSRVNfUEFTU1dPUkQSC3N1cGVyc2VjcmV0GgZPcGFxdWUaACIA"
    }
  ],
  "count": 1
}

echo $(etcdctl -w json get  /registry/secrets/default/pgsql | jq -r '.kvs[0].key' | base64 -d)
/registry/secrets/default/pgsql

etcdctl -w json get  /registry/secrets/default/pgsql | jq -r '.kvs[0].value' | base64 -d | head --byte
=10 | xxd
00000000: 6b38 7300 0a0c 0a02 7631                 k8s.....v1

etcdctl -w json get  /registry/secrets/default/pgsql | jq -r '.kvs[0].value' | base64 -d | tail --bytes=+5 | protoc --decode_raw

1 {
  1: "v1"
  2: "Secret"
}
2 {
  1 {
    1: "pgsql"
    2: ""
    3: "default"
    4: ""
    5: "e769ef47-0744-47b2-9025-3d5c59a5b0cf"
    6: ""
    7: 0
    8 {
      1: 1741626659
      2: 0
    }
    12 {
      1: "kubectl.kubernetes.io/last-applied-configuration"
      2: "{\"apiVersion\":\"v1\",\"kind\":\"Secret\",\"metadata\":{\"annotations\":{},\"name\":\"pgsql\",\"namespace\":\"default\"},\"stringData\":{\"POSTGRES_PASSWORD\":\"supersecret\"}}\n"
    }
    17 {
      1: "kubectl-client-side-apply"
      2: "Update"
      3: "v1"
      4 {
        1: 1741626659
        2: 0
      }
      6: "FieldsV1"
      7 {
        1: "{\"f:data\":{\".\":{},\"f:POSTGRES_PASSWORD\":{}},\"f:metadata\":{\"f:annotations\":{\".\":{},\"f:kubectl.kubernetes.io/last-applied-configuration\":{}}},\"f:type\":{}}"
      }
      8: ""
    }
  }
  2 {
    1: "POSTGRES_PASSWORD"
    2: "supersecret"
  }
  3: "Opaque"
}
3: ""
4: ""

