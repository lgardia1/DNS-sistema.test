Vagrant.configure("2") do |config|

  config.vm.box = "debian/bookworm64"

  # This if you have plugins vagrant-vbguest avoid a eternal loading time 
  config.vbguest.auto_update = false

  config.vm.define "earth" do |earth|
    earth.vm.network "private_network", ip: "192.168.57.103"

    earth.vm.hostname = "tierra.sistema.test"

    earth.vm.provision "shell", inline: <<-SHELL
        # Here put programs provision
        apt update -y
        apt-get install -y dnsutils bind9
    SHELL

    earth.vm.provision "shell", inline: <<-SHELL
      # Here put file provision
      cp -v /vagrant/config/resolution/resolv.conf /etc/ # Save DNS IPv4 resolution
      cp -v /vagrant/config/default/named /etc/default/named # Save IPv4 listening only enable  
      cp -v /vagrant/config/options/named.conf.options /etc/bind/named.conf.options # Save DNS options 
      cp -v /vagrant/config/earth/named.conf.local /etc/bind/named.conf.local # Save direct zone and inverted zone
      mkdir -p /etc/bind/zones # If exist make directory zones
      cp -v /vagrant/config/earth/db.sistema.test /etc/bind/zones/db.sistema.test # Save databse direct zone
      cp -v /vagrant/config/earth/db.192.168.57 /etc/bind/zones/db.192.168.57 # Save databse inverted zone
      systemctl restart bind9 # Restart configuration
    SHELL
  end

  config.vm.define "venus" do |venus|
    venus.vm.network "private_network", ip: "192.168.57.102"

    venus.vm.hostname = "venus.sistema.test"

    venus.vm.provision "shell", inline: <<-SHELL
      # Here put programs provision
      apt update -y
      apt-get install -y dnsutils bind9
    SHELL

    
    venus.vm.provision "shell", inline: <<-SHELL
      #Here put file provision
      cp -v /vagrant/config/resolution/resolv.conf /etc/ # Save DNS IPv4 resolution
      cp -v /vagrant/config/default/named /etc/default/ # Save IPv4 listening only enable
      cp -v /vagrant/config/options/named.conf.options /etc/bind/ # Save DNS options 
      cp -v /vagrant/config/venus/named.conf.local /etc/bind/ # Save direct zone and inverted zone
      mkdir -p /etc/bind/zones # If exist make directory zones
      touch /etc/bind/zones/db.sistema.test # Create empty direct zone file  
      touch /etc/bind/zones/db.192.168.57 # Create empty direct zone file  
      systemctl restart bind9 # Restart configuration
    SHELL

  end

end
