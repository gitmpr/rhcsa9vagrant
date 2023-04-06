$hostsfile = <<-SCRIPT
echo "192.168.99.10   anode" >> /etc/hosts
echo "192.168.99.11   cathode" >> /etc/hosts
SCRIPT

$anode_repo_setup = <<-SCRIPT
mkdir -p /mnt/repo
mount -t iso9660 -o ro /dev/sr0 /mnt/repo

dnf config-manager --set-disabled epel

cat << EOF > /etc/yum.repos.d/BaseOS.repo
[BaseOS]
name=BaseOS repo
baseurl=file:///mnt/repo/BaseOS
EOF

cat << EOF > /etc/yum.repos.d/AppStream.repo
[AppStream]
name=AppStream repo
baseurl=file:///mnt/repo/AppStream
EOF

dnf install -y httpd policycoreutils-python-utils

cat << EOF > /etc/httpd/conf.d/repo.conf
<Directory /mnt/repo>
    AllowOverride None
    Options +Indexes
    Require all granted
</Directory>

Alias /repo /mnt/repo
EOF

semanage fcontext -a -t httpd_sys_content_t "/mnt/repo(/.*)?"
restorecon -Rv /mnt/repo

firewall-cmd --permanent --add-service http
firewall-cmd --reload

systemctl enable --now httpd
SCRIPT

$cathode_repo_setup = <<-SCRIPT

dnf config-manager --set-disabled epel

cat << EOF > /etc/yum.repos.d/BaseOS.repo
[BaseOS]
name=BaseOS repo
baseurl=http://anode/repo/BaseOS
EOF

cat << EOF > /etc/yum.repos.d/AppStream.repo
[AppStream]
name=AppStream repo
baseurl=http://anode/repo/AppStream
EOF
SCRIPT



Vagrant.configure("2") do |config|

  # Define virtualbox as the provider
  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
  end

  # Create the first VM called anode
  config.vm.define :anode do |anode|
    anode.vm.box = "generic/rhel9"
    config.vm.box_version = "4.2.2"
    anode.vm.hostname = "anode"

#    This would set up a NAT network and forward a port through it, but also let the VM have internet access
#    anode.vm.network "forwarded_port", guest: 80, host: 8001

    anode.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = "2"

    end

    #anode.vm.network "private_network", ip: "192.168.99.10", virtualbox__intnet: true
    #anode.vm.network "hostonly", ip: "192.168.99.10", virtualbox__intnet: true
    anode.vm.network :private_network, ip: "192.168.99.10"
    ###anode.vm.network :hostonly, "192.168.99.10", :netmask => "255.255.255.0"
    #config.vm.network :hostonly, "192.168.50.4"
    
    # Add a DVD drive with the rhel 9.0 iso inserted
    anode.vm.provider "virtualbox" do |vb|
#      vb.customize ['storagectl', :id, '--name', 'IDE', '--add', 'ide']
      vb.customize ['storageattach', :id, '--storagectl', 'IDE Controller', '--port', '1', '--device', '0', '--type', 'dvddrive', '--medium', 'rhel-baseos-9.0-x86_64-dvd.iso']
    end
    anode.vm.provision "hostsfile setup", type: "shell", inline: $hostsfile
    anode.vm.provision "anode repo setup", type: "shell", inline: $anode_repo_setup

  end

  # Create the second VM called cathode
  config.vm.define :cathode do |cathode|
    cathode.vm.box = "generic/rhel9"
    config.vm.box_version = "4.2.2"
    cathode.vm.hostname = "cathode"

    #node.vm.network "forwarded_port", guest: 22, host: 2201

    cathode.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = "2"
    end
    cathode.vm.provision "hostsfile setup", type: "shell", inline: $hostsfile
    cathode.vm.provision "cathode repo setup", type: "shell", inline: $cathode_repo_setup
    cathode.vm.provision "disable epel setup", type: "shell", inline: <<-SHELL
      dnf config-manager --set-disabled epel
    SHELL

    #cathode.vm.network "private_network", ip: "192.168.99.11", virtualbox__intnet: true
    #cathode.vm.network "hostonly", ip: "192.168.99.11", virtualbox__intnet: true
    cathode.vm.network :private_network, ip: "192.168.99.11"
    ###cathode.vm.network :hostonly, "192.168.99.11", :netmask => "255.255.255.0"
  end
end

