VAGRANTFILE_API_VERSION = "2"
ROOT_IP="192.168.50.11"
N_COMP_NODES=1
N_CORES=2
LOCAL_NFS="/home/simon/work/nfs"



script = <<-SHELL
     set -x
     if [[ ! -e /etc/.provisioned ]]; then
     mkdir -p /root/.ssh
     cp ~vagrant/.ssh/authorized_keys /root/.ssh

     yum update -y
     yum install gcc gcc-c++ -y
     yum install ksh git -y

     rm /etc/hosts
     echo "ROOT_IP0 master" >>/etc/hosts 
     for i in {1..N_COMP_NODES}; do
       ip="ROOT_IP$((i))"
     echo "$ip compute-node-$i node$i" >> /etc/hosts
     done
     # we only generate the key on one of the nodes
      if [[ ! -e /vagrant/id_rsa ]]; then
        ssh-keygen -t rsa -f /vagrant/id_rsa -N ""
      fi
      install -m 600 -o vagrant -g vagrant /vagrant/id_rsa /home/vagrant/.ssh/
      # the extra 'echo' is needed because Vagrant inserts its own key without #a
      # newline at the end
      (echo; cat /vagrant/id_rsa.pub) >> /home/vagrant/.ssh/authorized_keys
      echo "StrictHostKeyChecking no" >> /home/vagrant/.ssh/config
      chown vagrant:vagrant /home/vagrant/.ssh/config

      echo "export PATH=\$PATH:/usr/lib64/mpich/bin" >> /home/vagrant/.bashrc
      echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:/usr/lib64/mpich/lib" >> /home/vagrant/.bashrc
      touch /etc/.provisioned
      fi

SHELL




Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
#  config.vm.box = "ubuntu/xenial64"
  #  config.vm.box = "centos/7"
  config.vm.box="cedadev/centos6"
  config.vm.provider "virtualbox" do |v|
    v.cpus = 2
  end
  config.vm.network "private_network", type: "dhcp"
  (0..N_COMP_NODES).each do |i|
    if i ==0 
      vm_name="master"
    else
      vm_name="compute-node-#{i}"
    end
    config.vm.define vm_name do |co|
      co.vm.hostname = "#{vm_name}"
      co.vm.network "private_network",ip: ROOT_IP+"#{i}",virtualbox__intnet: "clusternet"
    end
  end
  config.vm.synced_folder LOCAL_NFS, "/vagrant", type: "nfs", mount_options: ['rw', 'vers=3', 'tcp', 'fsc', 'actimeo=1']

  script.gsub! 'N_COMP_NODES', N_COMP_NODES.to_s
  script.gsub! 'ROOT_IP', ROOT_IP.to_s
  config.vm.provision "shell", inline: script
  
  config.vm.provision :ansible do |ans|
      ans.force_remote_user = false
      ans.extra_vars = { n_comp_nodes: N_COMP_NODES, n_cores: N_CORES}
      ans.playbook = "playbook/vmcluster.yml"
    end

end
