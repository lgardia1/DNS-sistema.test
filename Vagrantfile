Vagrant.configure("2") do |config|

  config.vm.box = "debian/bookworm64"

  config.vbguest.auto_update = false

  config.vm.define "earth" do |earth|
    earth.vm.network "private_network", ip: "192.168.57.103"

    earth.vm.hostname = "tierra.sistema.test"

    earth.vm.provision "shell", inline: <<-SHELL
       apt update -y
       apt-get install -y dnsutils bind9
     SHELL
  end

  config.vm.define "venus" do |venus|
    venus.vm.network "private_network", ip: "192.168.57.102"

    venus.vm.hostname = "venus.sistema.test"

    venus.vm.provision "shell", inline: <<-SHELL
      apt update -y
      apt-get install -y dnsutils bind9
  SHELL
  end
end
