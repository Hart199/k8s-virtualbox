$num_nodes = 2
$vm_cpus = 2
$vm_memory = 2048
$vm_box = "centos/7"
$vm_box_version = "1802.01"
#$vm_box = "Iorek/k8svirtualbox"
#$vm_box_version = "1.9.5"
$k8s_version = "v1.9.5"
$k8s_cluster_ip_tpl = "192.168.33.%s"
$k8s_master_ip = $k8s_cluster_ip_tpl % "10"
$vm_name_tpl = "vg-k8s-%s"

Vagrant.configure("2") do |config|
	config.vm.define "master", primary: true do |master|
		master.vm.box = $vm_box
		master.vm.box_version = $vm_box_version

		master.vm.box_check_update = false
	  
		master.vm.hostname = $vm_name_tpl % "master"
	  
		master.vm.network "private_network", ip: $k8s_master_ip

		master.vm.provider "virtualbox" do |vb|
			vb.name = $vm_name_tpl % "master"
			vb.memory = $vm_memory
			vb.cpus = $vm_cpus
			vb.gui = false
			
			# On VirtualBox, we don't have guest additions or a functional vboxsf，
			# so tell Vagrant that so it can be smarter.
			#vb.check_guest_additions = false
			#vb.functional_vboxsf     = false
			
			#vb.customize ['modifyvm', :id, '--nic1', 'hostonly', '--nic2', 'nat']
			#vb.customize ['modifyvm', :id, '--natpf2', 'ssh,tcp,127.0.0.1,2222,,22']
		end
		
		master.vm.provision :shell, :path => 'preflight.sh', :args => [$k8s_version]
		
		master.vm.provision "shell", :args => [$k8s_master_ip, $k8s_version], inline: <<-SHELL
			# allow root to run kubectl
			echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
			source /etc/profile
			
			ip addr show

			# initialize k8s master node
			kubeadm init --apiserver-advertise-address $1 --kubernetes-version $2 --pod-network-cidr 10.244.0.0/16 > ~/install.log

			# grep the join command
			sed -n '/kubeadm join/p' ~/install.log > ~/join.txt
			cp ~/join.txt /home/vagrant/join.txt

			# install flannel
			kubectl apply -f ~/k8s-utils/yaml/kube-flannel-vagrant.yml
		SHELL
	end
	
	(1..$num_nodes).each do |i|
		config.vm.define "node#{i}" do |node|
			node.vm.box = $vm_box
			node.vm.box_version = $vm_box_version

			node.vm.box_check_update = false
		  
			node.vm.hostname = $vm_name_tpl % "node-#{i}"
		  
			node.vm.network "private_network", ip: $k8s_cluster_ip_tpl % "#{i+10}"

			node.vm.provider "virtualbox" do |vb|
				vb.name = $vm_name_tpl % "node-#{i}"
				vb.memory = $vm_memory
				vb.cpus = $vm_cpus
				vb.gui = false
				
				#vb.customize ['modifyvm', :id, '--nic1', 'hostonly', '--nic2', 'nat']
				#vb.customize ['modifyvm', :id, '--natpf2', 'ssh,tcp,127.0.0.1,2222,,22']
			end
			
			node.vm.provision "shell", path: "preflight.sh", :args => [$k8s_version]
			
			node.vm.provision "shell", inline: <<-SHELL
				echo "initialize node-#{i}"
			SHELL
			
			node.vm.provision "shell", path: "join-cluster.sh", :args => [$k8s_master_ip]
		end
	end
end
