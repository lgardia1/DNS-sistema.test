Vagrant.configure("2") do |config|

  config.vm.box = "debian/bookworm64"

  # This if you have plugins vagrant-vbguest avoid a eternal loading time 
  config.vbguest.auto_update = false

  config.vm.define "earth" do |earth|
    earth.vm.network "private_network", ip: "192.168.57.103"

    earth.vm.hostname = "tierra.sistema.test"

    earth.vm.provision "shell", inline: <<-SHELL
        # Here put program provision
        apt update -y
        apt-get install -y dnsutils bind9
    SHELL

    earth.vm.provision "shell", inline: <<-SHELL
      # Here put file provision
      cp /vagrant/config/earth/named /etc/default/named
      cp /vagrant/config/earth/named.conf.options /etc/bind/named.conf.options
      cp /vagrant/config/earth/named.conf.local /etc/bind/named.conf.local 
      cp /vagrant/config/earth/db.sistema.test /etc/bind/zones/db.sistema.test
    SHELL
  end

  config.vm.define "venus" do |venus|
    venus.vm.network "private_network", ip: "192.168.57.102"

    venus.vm.hostname = "venus.sistema.test"

    venus.vm.provision "shell", inline: <<-SHELL
      # Here put program provision
      apt update -y
      apt-get install -y dnsutils bind9
    SHELL

    
    venus.vm.provision "shell", inline: <<-SHELL
      #Here put file provision
    SHELL
  end
end
