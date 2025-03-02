windows:
https://www.virtualbox.org/wiki/Downloads

wsl:
apt install vagrant -y
or
chatgpt advice:
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
apt update && sudo apt install vagrant -y
hashicorp advice:
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant

----------------
apt install ruby-full build-essential -y
vagrant plugin install vagrant-wsl2
vagrant plugin install vagrant-wsl2 --plugin-clean-sources --plugin-source https://rubygems.org/
-------------------------
apt install ansible
apt install pipx
pipx ensurepath
pipx install pywinrm

nano ~/.bashrc
export VAGRANT_WSL_ENABLE_WINDOWS_ACCESS="1"
export PATH="$PATH:/mnt/c/Program Files/Oracle/VirtualBox"


windows:
vagrant up
vagrant ssh host01

Add network interface in VirtualBox
https://medium.com/codemonday/ssh-to-virtualbox-host-only-adapter-interface-from-wsl2-machine-f17fc0978edd
in virtualbox:
ip link
ip link set enp0s8 up
ip addr add 192.168.56.11/24 dev enp0s8
ip addr

in WSL:

cp /mnt/c/Nauka/examples/chapter-02/.vagrant/machines/host01/virtualbox/private_key ~/.ssh/private_key
chmod 600 ~/.ssh/private_key

ssh -i ~/.ssh/private_key vagrant@192.168.56.11 -p 22


mv /mnt/c/examples/chapter-02/ansible.cfg ~/.ansible.cfg

fix path in ansible.cfg
[defaults]
collections_path = /mnt/c/Nauka/examples/setup/collections
inventory = /mnt/c/Nauka/examples/setup/ec2-inventory
roles_path = /mnt/c/Nauka/examples/setup/roles

[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no
---------------------------

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  cluster.each do |name, data|
    config.vm.define name do |host|
      host.vm.hostname = name
      host.vm.network "private_network", ip: "#{data[:ip]}"
      host.vm.provider :virtualbox do |vb, override|
        vb.cpus = data[:cpus]
        vb.memory = data[:mem]
      end
      host.ssh.private_key_path = "~/.ssh/private_key"
      host.ssh.forward_agent = true
    end
  end
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yaml"
    ansible.groups = groups
    ansible.inventory_path = "../setup/ec2-inventory/inventory"
  end
  config.vm.provision "extra", type: "ansible", run: "never" do |ansible|
    ansible.playbook = "extra.yaml"
    ansible.groups = groups
    ansible.inventory_path = "../setup/ec2-inventory/inventory"
  end
  config.vm.provision "test", type: "ansible", run: "never" do |ansible|
    ansible.playbook = "test.yaml"
    ansible.groups = groups
    ansible.inventory_path = "../setup/ec2-inventory/inventory"
  end
end
-----------------------------------------------------------
setup/ec2-inventory/inventory
[remote]
host01 ansible_host=192.168.56.11 ansible_port=22 ansible_user=vagrant ansible_private_key_file=~/.ssh/private_key

cd /mnt/c/Nauka/examples/chapter-02/
vagrant up
vagrant status
vagrant provision
vagrant provision --provision-with extra