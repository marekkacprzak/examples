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

https://medium.com/@priyantha.getc/step-by-step-guide-to-creating-a-kubernetes-cluster-on-ubuntu-22-04-using-containerd-runtime-0ead53a8d273

/etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

pt-get install -y apt-transport-https ca-certificates curl \
  gpg gnupg2 software-properties-common


mkdir -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list



