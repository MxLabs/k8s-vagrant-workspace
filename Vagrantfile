# -*- mode: ruby -*-
# vi: set ft=ruby :

$master = <<-SHELL
  IPADDR=$(ip a show enp0s8 | grep inet | grep -v inet6 | awk '{print $2}' | cut -f1 -d/)
  HOSTNAME=$(hostname -s)
  kubeadm init --apiserver-advertise-address=$IPADDR --apiserver-cert-extra-sans=$IPADDR --node-name $HOSTNAME --pod-network-cidr=10.244.0.0/16
  sudo --user=vagrant mkdir -p /home/vagrant/.kube
  cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
  chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config
  export KUBECONFIG=/etc/kubernetes/admin.conf
  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
  kubeadm token create --print-join-command > /etc/kubeadm_join_cmd.sh
  chmod +x /etc/kubeadm_join_cmd.sh
  sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
  systemctl restart sshd

SHELL


$node = <<-SHELL
  apt-get install -y sshpass
  sshpass -p "vagrant" scp -o StrictHostKeyChecking=no vagrant@192.168.21.11:/etc/kubeadm_join_cmd.sh .
  sh ./kubeadm_join_cmd.sh
SHELL


Vagrant.configure("2") do |config|
    (1..4).each do |i|
        if i == 1 then
            vm_name = "master"
        else
            vm_name = "node#{i-1}"
        end
        config.vm.define vm_name do |client|
            client.vm.box ="mxlabs/k8s-base-xenial64"
            client.vm.hostname = vm_name
            client.vm.network "private_network", ip: "192.168.21.#{i+10}"
            client.vm.provider "virtualbox" do |v|
                v.gui = false      
                v.name = vm_name  
                if i == 1 then
                    v.cpus = 2
                    v.memory = 1024
                else
                    v.cpus = 1
                    v.memory = 1024
                end
            end
            if i == 1 then
                client.vm.provision "shell", inline: $master
            else
                client.vm.provision "shell", inline: $node
            end
        end
    end
end