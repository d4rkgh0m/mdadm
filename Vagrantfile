# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
    :otuslinux => {
        :box_name => "centos/7",
        :ip_addr => '192.168.56.101',
        :disks => {
            :sata1 => {
                :dfile => './sata1.vdi',
                :size => 250,
                :port => 1
            },
            :sata2 => {
                :dfile => './sata2.vdi',
                :size => 250,
                :port => 2
            },
            :sata3 => {
                :dfile => './sata3.vdi',
                :size => 250,
                :port => 3
            },
            :sata4 => {
                :dfile => './sata4.vdi',
                :size => 250,
                :port => 4
            },
            :sata5 => {
                :dfile => './sata5.vdi',
                :size => 250,
                :port => 5
            },
            :sata6 => {
                :dfile => './sata6.vdi',
                :size => 250,
                :port => 6
            },

        }
    },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s
          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            	  vb.customize ["modifyvm", :id, "--memory", "1024"]
                  needsController = false
		  boxconfig[:disks].each do |dname, dconf|
			  unless File.exist?(dconf[:dfile])
				vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                needsController =  true
                          end
		  end
                  if needsController == true
                     vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                     boxconfig[:disks].each do |dname, dconf|
                         vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                     end
                  end
          end
 	  box.vm.provision "shell", inline: <<-SHELL
	      mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
	      yum install -y mdadm smartmontools hdparm gdisk
  	  SHELL
        
      box.vm.provision "shell" , path: "scripts/createraid.sh"

      end
  end
end
