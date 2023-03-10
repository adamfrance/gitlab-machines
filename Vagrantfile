Vagrant.configure(2) do |config|

	# before you must install these plugins to speed up vagrant provisionning
  # vagrant plugin install vagrant-faster
  # vagrant plugin install vagrant-cachier
	# Set some variables
  etcHosts = ""
  gitlabUrl = ""

	case ARGV[0]
		when "provision", "up"
  	print "Which url for your gitlab ?\n"
  	gitlabUrl = STDIN.gets.chomp
  	print "\n"
  else
		#do nothing
  end
	# some settings for common server (not for haproxy)
  common = <<-SHELL
  sudo apt update -qq 2>&1 >/dev/null
  sudo apt install -y -qq git vim tree net-tools telnet git python3-pip sshpass nfs-common 2>&1 >/dev/null
  sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
  sudo systemctl restart sshd
  SHELL

  docker = <<-SHELL
  curl -fsSL https://get.docker.com -o get-docker.sh 2>&1 
  sudo sh get-docker.sh 2>&1 >/dev/null
  sudo usermod -aG docker vagrant
  sudo service docker start
  sudo echo "autocmd filetype yaml setlocal ai ts=2 sw=2 et" > /home/vagrant/.vimrc
  sudo systemctl restart sshd
  SHELL
	# set vagrant image
	config.vm.box = "ubuntu/focal64"
	config.vm.box_url = "ubuntu/focal64"

	# set servers list and their parameters
	NODES = [
  	{ :hostname => "gitlab", :ip => "192.168.12.40", :cpus => 4, :mem => 6144, :type => "gitlab" },
  	{ :hostname => "runner1", :ip => "192.168.12.41", :cpus => 4, :mem => 4096, :type => "runner" },
  	{ :hostname => "runner2", :ip => "192.168.12.42", :cpus => 4, :mem => 4096, :type => "runner" },
  	{ :hostname => "target1", :ip => "192.168.12.43", :cpus => 2, :mem => 2048, :type => "target" },
  	{ :hostname => "target2", :ip => "192.168.12.44", :cpus => 2, :mem => 2048, :type => "target" }
	]

	# define /etc/hosts for all servers
  NODES.each do |node|
			etcHosts += "echo '" + node[:ip] + "   " + node[:hostname] + "'>> /etc/hosts" + "\n"
    if node[:type] == "gitlab"
      etcHosts += "echo '" + node[:ip] + "   " + gitlabUrl + " registry." + gitlabUrl + "' >> /etc/hosts" + "\n"
    end
  end #end NODES

	# run installation
  NODES.each do |node|
    config.vm.define node[:hostname] do |cfg|
			cfg.vm.hostname = node[:hostname]
      cfg.vm.network "private_network", ip: node[:ip]
      cfg.vm.provider "virtualbox" do |v|
				v.customize [ "modifyvm", :id, "--cpus", node[:cpus] ]
        v.customize [ "modifyvm", :id, "--memory", node[:mem] ]
        v.customize [ "modifyvm", :id, "--natdnshostresolver1", "on" ]
        v.customize [ "modifyvm", :id, "--natdnsproxy1", "on" ]
        v.customize [ "modifyvm", :id, "--name", node[:hostname] ]
				v.customize [ "modifyvm", :id, "--ioapic", "on" ]
        v.customize [ "modifyvm", :id, "--nictype1", "virtio" ]
      end #end provider
			
			#for all
      cfg.vm.provision :shell, :inline => etcHosts
			cfg.vm.provision :shell, :inline => common

      if node[:type] != "target"
			cfg.vm.provision :shell, :inline => docker
			end

			#for gitlab
      if node[:type] == "gitlab"
				cfg.vm.provision :shell, :path => "install_gitlab.sh", :args => [gitlabUrl]
			end

			#for runner
      if node[:type] == "runner"
				#cfg.vm.provision :shell, :path => "install_runner.sh"
			end

    end # end config
  end # end nodes

end 


