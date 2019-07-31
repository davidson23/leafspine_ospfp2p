# -*- mode: ruby -*-
# vi: set ft=ruby :


S1 = [
    { "mac"=>"080027FFBB01", "intnet"=>"swp1_S1L1", "intfIdx"=>1},
    { "mac"=>"080027FFBB02", "intnet"=>"swp3_S1L2", "intfIdx"=>2}
  ]
  
S2 = [
    { "mac"=>"080027FFBB03", "intnet"=>"swp1_S2L2", "intfIdx"=>1},
    { "mac"=>"080027FFBB04", "intnet"=>"swp3_S2L1", "intfIdx"=>2}
  ]

L1 = [
    { "mac"=>"080027FFBB05", "intnet"=>"swp1_S1L1", "intfIdx"=>1},
    { "mac"=>"080027FFBB06", "intnet"=>"swp2_PC1", "intfIdx"=>2},
    { "mac"=>"080027FFBB07", "intnet"=>"swp3_S2L1", "intfIdx"=>3}
  ]
  
  
L2 = [
    { "mac"=>"080027FFBB08", "intnet"=>"swp1_S2L2", "intfIdx"=>1},
    { "mac"=>"080027FFBB09", "intnet"=>"swp2_PC2", "intfIdx"=>2},
    { "mac"=>"080027FFBB0A", "intnet"=>"swp3_S1L2", "intfIdx"=>3}
  ]
  
PC1 = [
    { "mac"=>"080027FFBB0B", "intnet"=>"swp2_PC1", "intfIdx"=>1}
  ]
  
PC2 = [
    { "mac"=>"080027FFBB0C", "intnet"=>"swp2_PC2", "intfIdx"=>1}
  ]
Vagrant.require_version ">= 2.0.2"

# Fix for Older versions of Vagrant to Grab Images from the Correct Location
unless Vagrant::DEFAULT_SERVER_URL.frozen?
  Vagrant::DEFAULT_SERVER_URL.replace('https://vagrantcloud.com')
end

$script = <<-SCRIPT
if grep -q -i 'cumulus' /etc/lsb-release &> /dev/null; then
    echo "### RUNNING CUMULUS EXTRA CONFIG ###"
    source /etc/lsb-release
    if [ -z /etc/app-release ]; then
        echo "  INFO: Detected NetQ TS Server"
        source /etc/app-release
        echo "  INFO: Running NetQ TS Appliance Version $APPLIANCE_VERSION"
    else
        if [[ $DISTRIB_RELEASE =~ ^2.* ]]; then
            echo "  INFO: Detected a 2.5.x Based Release"

            echo "  adding fake cl-acltool..."
            echo -e "#!/bin/bash\nexit 0" > /usr/bin/cl-acltool
            chmod 755 /usr/bin/cl-acltool

            echo "  adding fake cl-license..."
            echo -e "#!/bin/bash\nexit 0" > /usr/bin/cl-license
            chmod 755 /usr/bin/cl-license

            echo "  Disabling default remap on Cumulus VX..."
            mv -v /etc/init.d/rename_eth_swp /etc/init.d/rename_eth_swp.backup

            echo "### Rebooting to Apply Remap..."
        elif [[ $DISTRIB_RELEASE =~ ^3.* ]]; then
            echo "  INFO: Detected a 3.x Based Release ($DISTRIB_RELEASE)"
            echo "### Disabling default remap on Cumulus VX..."
            mv -v /etc/hw_init.d/S10rename_eth_swp.sh /etc/S10rename_eth_swp.sh.backup &> /dev/null
            echo "  INFO: Detected Cumulus Linux v$DISTRIB_RELEASE Release"
            if [[ $DISTRIB_RELEASE =~ ^3.[1-9].* ]]; then
                echo "### Fixing ONIE DHCP to avoid Vagrant Interface ###"
                echo "     Note: Installing from ONIE will undo these changes."
                mkdir /tmp/foo
                mount LABEL=ONIE-BOOT /tmp/foo
                sed -i 's/eth0/eth1/g' /tmp/foo/grub/grub.cfg
                sed -i 's/eth0/eth1/g' /tmp/foo/onie/grub/grub-extra.cfg
                umount /tmp/foo
            fi
            if [[ $DISTRIB_RELEASE =~ ^3.2.* ]]; then
                if [[ $(grep "vagrant" /etc/netd.conf | wc -l ) == 0 ]]; then
                    echo "### Giving Vagrant User Ability to Run NCLU Commands ###"
                    sed -i 's/users_with_edit = root, cumulus/users_with_edit = root, cumulus, vagrant/g' /etc/netd.conf
                    sed -i 's/users_with_show = root, cumulus/users_with_show = root, cumulus, vagrant/g' /etc/netd.conf
                fi
            elif [[ $DISTRIB_RELEASE =~ ^3.[3-9].* ]]; then
                echo "### Giving Vagrant User Ability to Run NCLU Commands ###"
                adduser vagrant netedit
                adduser vagrant netshow
            fi
            echo "### Disabling ZTP service..."
            systemctl stop ztp.service
            ztp -d 2>&1
            echo "### Resetting ZTP to work next boot..."
            ztp -R 2>&1
            ztp -i 2>&1
        fi
    fi
fi
echo "### DONE ###"
#echo "### Rebooting Device to Apply Remap..."
#nohup bash -c 'sleep 10; shutdown now -r "Rebooting to Remap Interfaces"' &
SCRIPT


Vagrant.configure("2") do |config|

  config.vm.provider "virtualbox" do |v|
    v.gui=false

  end

  
  ##### DEFINE VM for L1 #####
  config.vm.define "L1" do |device|
    
    device.vm.hostname = "L1" 
    
    device.vm.box = "./cumulus-linux-3.7.6-vx-amd64-vbox.box"
    device.vm.provider "virtualbox" do |v|
      v.name = "L1"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 1024
    end
    
    device.vm.synced_folder ".", "/vagrant", disabled: true
	
    # NETWORK INTERFACES
      # link for swp1 --> to S1
      device.vm.network "private_network", virtualbox__intnet: "swp1_S1L1", auto_config: false , :mac => "080027FFBB05"
      # link for swp2 --> to PC1
      device.vm.network "private_network", virtualbox__intnet: "swp2_PC1", auto_config: false , :mac => "080027FFBB06"
      # link for swp3 --> to S2
      device.vm.network "private_network", virtualbox__intnet: "swp3_S2L1", auto_config: false , :mac => "080027FFBB07"
	  
    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all', "--nictype2", "virtio"]
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all', "--nictype3", "virtio"]
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all', "--nictype4", "virtio"]
      vbox.customize ["modifyvm", :id, "--nictype1", "virtio"]
    end
	
    # Run Any Platform Specific Code and Apply the interface Re-map
    #   (may or may not perform a reboot depending on platform)
    device.vm.provision :shell , :inline => $script

end

  ##### DEFINE VM for L2 #####
  config.vm.define "L2" do |device|
    
    device.vm.hostname = "L2" 
    
    device.vm.box = "./cumulus-linux-3.7.6-vx-amd64-vbox.box"
    device.vm.provider "virtualbox" do |v|
      v.name = "L2"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 1024
    end
    
    device.vm.synced_folder ".", "/vagrant", disabled: true
	
    # NETWORK INTERFACES
      # link for swp0 --> to S2
      device.vm.network "private_network", virtualbox__intnet: "swp1_S2L2", auto_config: false , :mac => "080027FFBB08"
      # link for swp2 --> to PC2
      device.vm.network "private_network", virtualbox__intnet: "swp2_PC2", auto_config: false , :mac => "080027FFBB09"
      # link for swp3 --> to S1
      device.vm.network "private_network", virtualbox__intnet: "swp3_S1L2", auto_config: false , :mac => "080027FFBB0A"
	  
    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all', "--nictype2", "virtio"]
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all', "--nictype3", "virtio"]
      vbox.customize ['modifyvm', :id, '--nicpromisc4', 'allow-all', "--nictype4", "virtio"]
      vbox.customize ["modifyvm", :id, "--nictype1", "virtio"]
    end
	
    # Run Any Platform Specific Code and Apply the interface Re-map
    #   (may or may not perform a reboot depending on platform)
    device.vm.provision :shell , :inline => $script

end

  ##### DEFINE VM for S1 #####
  config.vm.define "S1" do |device|
    
    device.vm.hostname = "S1" 
    
    device.vm.box = "./cumulus-linux-3.7.6-vx-amd64-vbox.box"
    device.vm.provider "virtualbox" do |v|
      v.name = "S1"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 1024
    end
    
    device.vm.synced_folder ".", "/vagrant", disabled: true
	
    # NETWORK INTERFACES
      # link for swp1 --> to L1
      device.vm.network "private_network", virtualbox__intnet: "swp1_S1L1", auto_config: false , :mac => "080027FFBB01"
      # link for swp3 --> to L2
      device.vm.network "private_network", virtualbox__intnet: "swp3_S1L2", auto_config: false , :mac => "080027FFBB02"

    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all', "--nictype2", "virtio"]
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all', "--nictype3", "virtio"]
      vbox.customize ["modifyvm", :id, "--nictype1", "virtio"]
    end
	
    # Run Any Platform Specific Code and Apply the interface Re-map
    #   (may or may not perform a reboot depending on platform)
    device.vm.provision :shell , :inline => $script

end

  ##### DEFINE VM for S2 #####
  config.vm.define "S2" do |device|
    
    device.vm.hostname = "S2" 
    
    device.vm.box = "./cumulus-linux-3.7.6-vx-amd64-vbox.box"
    device.vm.provider "virtualbox" do |v|
      v.name = "S2"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 1024
    end
    
    device.vm.synced_folder ".", "/vagrant", disabled: true
	
    # NETWORK INTERFACES
      # link for swp1 --> to L2
      device.vm.network "private_network", virtualbox__intnet: "swp1_S2L2", auto_config: false , :mac => "080027FFBB03"
      # link for swp3 --> to L1
      device.vm.network "private_network", virtualbox__intnet: "swp3_S2L1", auto_config: false , :mac => "080027FFBB04"

    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all', "--nictype2", "virtio"]
      vbox.customize ['modifyvm', :id, '--nicpromisc3', 'allow-all', "--nictype3", "virtio"]
      vbox.customize ["modifyvm", :id, "--nictype1", "virtio"]
    end
	
    # Run Any Platform Specific Code and Apply the interface Re-map
    #   (may or may not perform a reboot depending on platform)
    device.vm.provision :shell , :inline => $script

end

  ##### DEFINE VM for PC1 #####
  config.vm.define "PC1" do |device|
    
    device.vm.hostname = "PC1" 
    
    device.vm.box = "./cumulus-linux-3.7.6-vx-amd64-vbox.box"
    device.vm.provider "virtualbox" do |v|
      v.name = "PC1"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 1024
    end
    
    device.vm.synced_folder ".", "/vagrant", disabled: true
	
    # NETWORK INTERFACES
      # link for swp1 --> to L1
      device.vm.network "private_network", virtualbox__intnet: "swp2_PC1", auto_config: false , :mac => "080027FFBB0B"
	  
    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all', "--nictype2", "virtio"]
      vbox.customize ["modifyvm", :id, "--nictype1", "virtio"]
    end
	
    # Run Any Platform Specific Code and Apply the interface Re-map
    #   (may or may not perform a reboot depending on platform)
    device.vm.provision :shell , :inline => $script

end

  ##### DEFINE VM for PC2 #####
  config.vm.define "PC2" do |device|
    
    device.vm.hostname = "PC2" 
    
    device.vm.box = "./cumulus-linux-3.7.6-vx-amd64-vbox.box"
    device.vm.provider "virtualbox" do |v|
      v.name = "PC2"
      v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
      v.memory = 1024
    end
    
    device.vm.synced_folder ".", "/vagrant", disabled: true
	
    # NETWORK INTERFACES
      # link for swp1 --> to L2
      device.vm.network "private_network", virtualbox__intnet: "swp2_PC2", auto_config: false , :mac => "080027FFBB0C"
	  
    device.vm.provider "virtualbox" do |vbox|
      vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all', "--nictype2", "virtio"]
      vbox.customize ["modifyvm", :id, "--nictype1", "virtio"]
    end
	
    # Run Any Platform Specific Code and Apply the interface Re-map
    #   (may or may not perform a reboot depending on platform)
    device.vm.provision :shell , :inline => $script

end

end
