# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :otuslinux => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.101',
	:disks => {
		:sata1 => {
			:dfile => './sata1.vdi',
			:size => 250,
			:port => 1
		},
		:sata2 => {
                        :dfile => './sata2.vdi',
                        :size => 250, # Megabytes
			:port => 2
		},
                :sata3 => {
                        :dfile => './sata3.vdi',
                        :size => 250,
                        :port => 3
                },
                :sata4 => {
                        :dfile => './sata4.vdi',
                        :size => 250, # Megabytes
                        :port => 4
                }

	}

		
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

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
	      yum install -y mdadm smartmontools hdparm gdisk dosfstools
	      mdadm --create /dev/md10 --level=10 --raid-devices=4 /dev/sdb /dev/sdc /dev/sdd /dev/sde
	      mdadm --detail --verbose --scan > /etc/mdadm.conf
	      lsblk
	      sgdisk -Z /dev/md10
	      sgdisk -n 0:0:+95M -t 0:ef00 -c 0:"EFI partition" /dev/md10
	      sgdisk -n 0:0:+100M -t 0:8300 -c 0:"root partition" /dev/md10
	      sgdisk -n 0:0:+100M -t 0:8300 -c 0:"opt partition" /dev/md10
	      sgdisk -n 0:0:+100M -t 0:8300 -c 0:"var partition" /dev/md10
	      sgdisk -n 0:0:0 -t 0:8200 -c 0:"swap partition" /dev/md10
	      sgdisk -p /dev/md10
	      mkfs.vfat /dev/md10p1
	      mkfs.ext4 /dev/md10p2
	      mkfs.ext4 /dev/md10p3
	      mkfs.ext4 /dev/md10p4
	      mkswap /dev/md10p5
  	  SHELL

      end
  end
end

