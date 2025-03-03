windows:
vagrant up
add network interface 192.168.56.11-14
wsl:
mv /mnt/c/Nauka/examples1/chapter-06/.vagrant/machines/host01/virtualbox/private_key ~/.ssh/host01_private_key (4 key)
 
inventory:
[masters]
host01 ansible_host=192.168.56.11 ansible_port=22 ansible_user=vagrant ansible_private_key_file=~/.ssh/host01_private_key
host02 ansible_host=192.168.56.12 ansible_port=22 ansible_user=vagrant ansible_private_key_file=~/.ssh/host02_private_key
host03 ansible_host=192.168.56.13 ansible_port=22 ansible_user=vagrant ansible_private_key_file=~/.ssh/host03_private_key

[nodes]
host04 ansible_host=192.168.56.14 ansible_port=22 ansible_user=vagrant ansible_private_key_file=~/.ssh/host04_private_key

[remote:children]
masters
nodes
remote.yaml:
k8s_control_plane_endpoint: 192.168.56.11
k8s_initial_master: 192.168.56.11
k8s_cluster_ips:
  - 192.168.56.11
  - 192.168.56.12
  - 192.168.56.13
  - 192.168.56.14

ansible.cfg (in root)
[defaults]
collections_path = /mnt/c/Nauka/examples1/setup/collections
inventory = /mnt/c/Nauka/examples1/setup/ec2-inventory
roles_path = /mnt/c/Nauka/examples1/setup/roles

[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no

vagrant provision

ssh -i ~/.ssh/host01_private_key vagrant@192.168.56.11 -p 22

https://medium.com/@priyantha.getc/step-by-step-guide-to-creating-a-kubernetes-cluster-on-ubuntu-22-04-using-containerd-runtime-0ead53a8d273

k8s-all ufw disable
 k8s-all apt -y full-upgrade
k8s-all apt install systemd-timesyncd
k8s-all timedatectl set-ntp true
k8s-all timedatectl status
k8s-all "echo 'overlay' > /etc/modules-load.d/k8s.conf"
k8s-all "echo 'br_netfilter' >> /etc/modules-load.d/k8s.conf"
k8s-all "modprobe overlay"
k8s-all "modprobe br_netfilter"

k8s-all "echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.d/k8s.conf"
k8s-all "echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.d/k8s.conf"
k8s-all "echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.d/k8s.conf"
k8s-all "sysctl --system"

k8s-all "apt-get install -y apt-transport-https ca-certificates curl \
  gpg gnupg2 software-properties-common"

k8s-all "mkdir -m 755 /etc/apt/keyrings"
k8s-all "curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | \
 sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg"
k8s-all "echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list"
k8s-all "apt update"
k8s-all "apt install -y open-iscsi nfs-common"
source /opt/k8sver
k8s-all apt install -y kubelet=$K8SV kubeadm=$K8SV kubectl=$K8SV
systemctl status kubelet

k8s-all apt-mark hold kubelet kubeadm kubectl

 /usr/bin/kubeadm init --config /etc/kubernetes/kubeadm-init.yaml --upload-certs

-----------------------
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.56.11:6443 --token 1d8fb1.2875d52d62a3282d \
        --discovery-token-ca-cert-hash sha256:78037e51e921ba6eb498d272cafe68f867e0840bd8957c32501f5608cb8e0148 \
        --control-plane --certificate-key 5a7e07816958efb97635e9a66256adb1

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.56.11:6443 --token 1d8fb1.2875d52d62a3282d \
        --discovery-token-ca-cert-hash sha256:78037e51e921ba6eb498d272cafe68f867e0840bd8957c32501f5608cb8e0148
------------------------------------
host 02-04
kubeadm config images pull
cd /etc/kubernetes
kubeadm join --config kubeadm-join.yaml

-------------------
host 01
 kubectl get nodes
kubectl describe node host04
kubectl get nodes -o json | jq '.items[]|.metadata.name,.spec.taints[]'

"host01"
{
  "effect": "NoSchedule",
  "key": "node.kubernetes.io/not-ready"
}
{
  "effect": "NoExecute",
  "key": "node.kubernetes.io/not-ready",
  "timeAdded": "2025-03-03T09:56:32Z"
}
"host02"
{
  "effect": "NoSchedule",
  "key": "node.kubernetes.io/not-ready"
}
"host03"
{
  "effect": "NoSchedule",
  "key": "node.kubernetes.io/not-ready"
}
"host04"
{
  "effect": "NoSchedule",
  "key": "node.kubernetes.io/not-ready"
}

cd /etc/kubernetes/components/
source /opt/k8sver

---- siec

curl -L -O $calico_url
kubectl create -f tigera-operator.yaml
kubectl create -f custom-resources.yaml

-- pamiecmasowa

k8s-all systemctl enable --now iscsid

curl -LO $longhorn_url
kubectl create -f longhorn.yaml

curl -Lo ingress-controller.yaml $ingress_url
kubectl create -f ingress-controller.yaml

--- fix port
kubectl patch -n ingress-nginx \
 service/ingress-nginx-controller --patch-file ingress-patch.yaml

kubectl annotate -n ingress-nginx \
 ingressclass/nginx ingressclass.kubernetes.io/is-default-class="true"

-------- zatwierdzenie certow
kubectl get csr
kubectl certificate approve $(kubectl get csr --field-selector spec.signerName=kubernetes.io/kubelet-serving -o name)
---- serwer wskaznikow
curl -Lo metrics-server.yaml $metrics_url
