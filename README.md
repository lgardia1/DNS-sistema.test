# DNS-SISTEMA.TEST HomeWorks

## Index
- [DNS-SISTEMA.TEST HomeWorks](#dns-sistematest-homeworks)
  - [Index](#index)
    - [Introduction](#introduction)
  - [1. Configure VMs](#1-configure-vms)
    - [1.1 Create Vagrantfile](#11-create-vagrantfile)
    - [1.2 Edit Vagrantfile](#12-edit-vagrantfile)
    - [1.3 Start Vagrantfile](#13-start-vagrantfile)
  - [2. Configure Servers](#2-configure-servers)
    - [2.1 General configuration](#21-general-configuration)
      - [2.1.1 Listen only IPv4](#211-listen-only-ipv4)
      - [3.1.2 Resolution configuration](#312-resolution-configuration)
    - [2.2 Master Server (Earth)](#22-master-server-earth)
      - [2.2.1 Set Up general configuration](#221-set-up-general-configuration)
      - [2.2.2 Configuration Options:Enable dnssec-validation](#222-configuration-optionsenable-dnssec-validation)
      - [2.2.3 ACL Configuration: Recursive to specific networks](#223-acl-configuration-recursive-to-specific-networks)
      - [2.2.4 Configuration Options: Forward Server for Non-Authoritative Response](#224-configuration-options-forward-server-for-non-authoritative-response)
      - [2.2.5 Configure Local: DNS Zone](#225-configure-local-dns-zone)
      - [2.2.6 Create Zone Direct Database with alias and mail server](#226-create-zone-direct-database-with-alias-and-mail-server)
        - [What does "The server marte.sistema.test will act as the mail server for the domain sistema.test" mean?](#what-does-the-server-martesistematest-will-act-as-the-mail-server-for-the-domain-sistematest-mean)
      - [2.2.7 Create Zone Inverted Database](#227-create-zone-inverted-database)
    - [2.3 Slave Server (Venus)](#23-slave-server-venus)
      - [2.3.1 Set Up General Configuration](#231-set-up-general-configuration)
      - [2.3.2 Imports Configuration from Master Server](#232-imports-configuration-from-master-server)
      - [2.3.3 Configure DNS Zone](#233-configure-dns-zone)
  - [3 Check configuration](#3-check-configuration)
    - [3.1 Check configuration](#31-check-configuration)
    - [3.2 Check zones configuration](#32-check-zones-configuration)
    - [3.3 Check Name Resolution](#33-check-name-resolution)
    - [4 Save configuration](#4-save-configuration)
      - [Save files:](#save-files)
        - [General](#general)
        - [Files Master](#files-master)
        - [Files Slave](#files-slave)
      - [VagrantFile provision will be like this:](#vagrantfile-provision-will-be-like-this)
        - [Master](#master)
        - [Slave](#slave)
  - [Author](#author)


### Introduction

This documentation gives you the basic setup and configurations needed for setting up DNS servers using Bind9 with Vagrant.

---

## 1. Configure VMs

### 1.1 Create Vagrantfile
To start the configuration, you need to create a `Vagrantfile` using the following command:

```bash
vagrant init
```

### 1.2 Edit Vagrantfile
The **Vagrantfile** should be edited to create two VMs: one for the master DNS server (Earth) and the other for the slave DNS server (Venus). The configuration below have 2 provisions for each that installs the required packages like **bind9** and files configuration.

```ruby
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
      cp -v /vagrant/config/earth/named.conf.options /etc/bind/named.conf.options #  Save options of dns
      cp -v /vagrant/config/earth/named.conf.local /etc/bind/named.conf.local # Save direct zone and inverted zone
      mkdir -p /etc/bind/zones # If exist make directory zones
      cp -v /vagrant/config/earth/db.sistema.test /etc/bind/zones/db.sistema.test # Save databse direct zone
      cp -v /vagrant/config/earth/db.192.168.57 /etc/bind/zones/db.192.168.57 # Save databse inverted zone
      sudo systemctl restart bind9 # Restart configuration
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
      cp -v /vagrant/config/venus/named.conf.local /etc/bind/
      cp -v /vagrant/config/venus/named.conf.options /etc/bind/
      mkdir -p /etc/bind/zones # If exist make directory zones
      touch /etc/bind/zones/db.sistema.test # Create empty direct zone file  
      touch /etc/bind/zones/db.192.168.57 # Create empty direct zone file  
      sudo systemctl restart bind9 # Restart configuration
    SHELL

  end

end

```

### 1.3 Start Vagrantfile
To start the virtual machines, use the following command:
```bash
    vagrant up
```
---

## 2. Configure Servers

### 2.1 General configuration

#### 2.1.1 Listen only IPv4

Modify the startup options to only listen on IPv4 by adding **`-4`**:
```bash
OPTIONS="-u bind -4"
```

#### 3.1.2 Resolution configuration

Make sure that your resolution configuration is well done.
 
Edit *(`resolv.conf`)*:
```bash
    sudo nano /etc/resolv.conf
```
The *`nameserver`* will be the DNS server that resolves the names.
```bash
    sudo nameserver=192.168.57.103
```


### 2.2 Master Server (Earth)

#### 2.2.1 Set Up general configuration

[Here you can find out general configuration](#31-general-configuration)

#### 2.2.2 Configuration Options:Enable dnssec-validation

Edit the DNS options in the **named.conf.options** file to enable DNSSEC validation:
```bash
    sudo nano /etc/bind/named.conf.options
```

Modify it as follows:
```bash
    options {
            directory "/var/cache/bind";
            dnssec-validation yes;
            listen-on { any; }; // Listening any ip
            listen-on-v6 { none; }; // Disabling IPv6 listening
    };
```

#### 2.2.3 ACL Configuration: Recursive to specific networks

To limit recursive queries to specific networks, configure ACLs in the same **named.conf.options** file:
```bash
    sudo nano /etc/bind/named.conf.options
```

Modify it as follows:
```bash
    acl "allow_networks" {
            127.0.0.0/8; // Allow queries from localhost
            192.168.57.0/24; // Allow queries from the VM network
    };

    options {
            recursion yes;
            directory "/var/cache/bind";
            dnssec-validation yes;
            listen-on { any; };
            listen-on-v6 { none; };
    };
```

#### 2.2.4 Configuration Options: Forward Server for Non-Authoritative Response

Edit the DNS options in the **named.conf.options** file:
```bash
    sudo nano /etc/bind/named.conf.options
```
Modify it as follows:
```bash
    acl "allow_networks" {
            127.0.0.0/8; // Allow queries from localhost
            192.168.57.0/24; // Allow queries from the VM network
    };

    options {
            recursion yes;
            directory "/var/cache/bind";
            dnssec-validation yes;
            listen-on { any; };
            listen-on-v6 { none; };

            forwarders {
              208.67.222.222;  // Forward to OpenDNS
            };

            forward only;  // Indicate in case don't find resolution name, forward to another server
    };
```

#### 2.2.5 Configure Local: DNS Zone

Next, we define the DNS zone for tierra.sistema.test and his inverted zone in the **named.conf.local** file:
```bash
sudo nano /etc/bind/named.conf.local
```

Add the following zone configuration:
```bash
    zone "sistema.test" {
            type master;
            file "/etc/bind/zones/db.sistema.test";
};
      zone "57.168.192.in-addr.arpa" {
            type master;
            file "/etc/bind/zones/db.192.168.57";
};
```

Create the directory for zone files:
```bash
    sudo mkdir /etc/bind/zones
```

#### 2.2.6 Create Zone Direct Database with alias and mail server

**Create the DNS zone file (`db.sistema.test`)**:
```bash
    sudo touch /etc/bind/zones/db.sistema.test
```
**Then, edit the zone database file for (`sistema.test`)**:
```bash
    sudo nano /etc/bind/zones/db.sistema.test
```

In this file, add the necessary DNS records:
```bash
$TTL    604800
;
;
;dns an email
@       IN      SOA     tierra.sistema.test. admin.sistema.test. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

@       IN      NS      tierra.sistema.test.
;entry dns to ip
@       IN      A       192.168.57.103
venus   IN      A       192.168.57.102
marte   IN      A       192.168.57.104
mercurio    IN      A       192.168.57.101

; Alias (CNAME)
ns1     IN      CNAME   tierra.sistema.test.
ns2     IN      CNAME   venus.sistema.test.
mail    IN      CNAME   marte.sistema.test.

; Mail Exchange (MX)
@       IN      MX 10   marte ;Its unnecesary put .sistema.test.
```
##### What does "The server marte.sistema.test will act as the mail server for the domain sistema.test" mean?

This statement means that the server named `marte.sistema.test` will handle email processing for the domain **sistema.test**. In other words, any emails sent to addresses like `user@sistema.test` will be managed and processed by the server `marte.sistema.test`.

To configure this correctly, you need to add an **MX (Mail Exchange) record** to your DNS zone file. An MX record specifies the mail server responsible for receiving emails for a particular domain.

#### 2.2.7 Create Zone Inverted Database

**Create the DNS zone file (`db.192.168.57`)**:
```bash
    sudo touch zones/db.192.168.57
```
**Then, edit the zone database file for (`57.168.192.in-addr.arpa`)**:
```bash
    sudo nano zones/zones/db.192.168.57
```

In this file, add the necessary DNS records:
```bash
$TTL    604800
@       IN      SOA     tierra.sistema.test. admin.sistema.test. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

@       IN      NS      tierra.sistema.test.

; Registros PTR (inversos)
101     IN      PTR     mercurio.sistema.test.
102     IN      PTR     venus.sistema.test.
103     IN      PTR     tierra.sistema.test.
104     IN      PTR     marte.sistema.test.
```

### 2.3 Slave Server (Venus)

#### 2.3.1 Set Up General Configuration

[Here you can find out general configuration](#31-general-configuration)

#### 2.3.2 Imports Configuration from Master Server

With the provision we can imports part of config from the master server.

We will imports general configuration:
```ruby
    venus.vm.provision "shell", inline: <<-SHELL
      cp /vagrant/config/default/named /etc/default/
      cp /vagrant/config/resolution/resolv.conf /etc/
    SHELL
```



#### 2.3.3 Configure DNS Zone

So, we are going create two zone like master server, sistema.test and inverted.
```bash
    sudo nano /etc/bind/named.conf.local
```

```bash
zone "sistema.test" {
        type slave; # Slave
        file "/etc/bind/zones/db.sistema.test"; # Directory which server databse is save
        masters { 192.168.57.103; }; # IP DNS master
};

zone "57.168.192.in-addr.arpa" {
        type slave; # Slave
        file "/etc/bind/zones/db.192.168.57"; # Directory which server databse is save
        masters { 192.168.57.103; }; # IP DNS master
};
```

Check configuration
```bash
    sudo named-checkconf
```

Save configuration
```bash
    cp /etc/bind/named.conf.local /vagrant/config/venus/
```

## 3 Check configuration

### 3.1 Check configuration
```bash
    sudo named-checkconf
```
Don't return output if all is fine.

### 3.2 Check zones configuration
Use this layout to check configurations **`sudo named-checkzone yournamezone dbdirectory`**.


***Direct Zone***
```bash
    sudo named-checkzone tierra.sistema.test /etc/bind/zones/db.sistema.test
```

Output:
```bash
    zone tierra.sistema.test/IN: loaded serial 2
    OK
```

***Inverted Zone***
```bash
    sudo named-checkzone 57.168.192.in-addr.arpa /etc/bind/zones/db.192.162.57
```

Output:
```bash
    zone tierra.sistema.test/IN: loaded serial 2
    OK
```


### 3.3 Check Name Resolution
The previous step check the syntax, so we are going to check the dns resolution with **`nslookup`**.


***Direct Zone***
```bash
    nslookup tierra.sistema.test
```

Output:
```bash
    Server:         192.168.57.103   
    Address:        192.168.57.103#53

    Name:   tierra.sistema.test      
    Address: 192.168.57.103
```

***Inverted Zone***
```bash
    nslookup 192.168.57.102
```

Output:
```bash
    102.57.168.192.in-addr.arpa     name = venus.sistema.test.
```

> ⚠️ **Warning**: Make sure that your resolution configuration is well done.
> 
>    To have it well configured check **[Resolution Configuration](#312-resolution-configuration)**



### 4 Save configuration

#### Save files:

At the end of to set up your Master/Slave enviroment, save configuration of all files on your **guest** computer to the set the provision in the **VagrantFile**.

##### General
```bash
    cp /etc/resolv.conf /vagrant/config/resolution/
    cp /etc/default/named /vagrant/config/default/
```

##### Files Master
```bash
    cp /etc/bind/named.conf.options /vagrant/config/earth/
    cp /etc/bind/named.conf.local /vagrant/config/earth/
    cp /etc/bind/zones/db.sistema.test /vagrant/config/earth/
    cp /etc/bind/zones/db.192.168.57 /vagrant/config/earth/
```

##### Files Slave
```bash
    cp /etc/bind/named.conf.options /vagrant/config/venus/
    cp /etc/bind/named.conf.local /vagrant/config/venus/
    cp /etc/bind/zones/db.sistema.test /vagrant/config/venus/
    cp /etc/bind/zones/db.192.168.57 /vagrant/config/venus/
```

#### VagrantFile provision will be like this:

Make your sure of **create necessary directory** (like zones directory) and restart **bind9** 

##### Master
```bash
earth.vm.provision "shell", inline: <<-SHELL
  # Here put file provision
  cp -v /vagrant/config/resolution/resolv.conf /etc/ # Save DNS IPv4 resolution
  cp -v /vagrant/config/default/named /etc/default/named # Save IPv4 listening only enable  
  cp -v /vagrant/config/earth/named.conf.options /etc/bind/named.conf.options # Save DNS options
  cp -v /vagrant/config/earth/named.conf.local /etc/bind/named.conf.local # Save direct zone and inverted zone
  mkdir -p /etc/bind/zones # If exist make directory zones
  cp -v /vagrant/config/earth/db.sistema.test /etc/bind/zones/db.sistema.test # Save databse direct zone
  cp -v /vagrant/config/earth/db.192.168.57 /etc/bind/zones/db.192.168.57 # Save databse inverted zone
  sudo systemctl restart bind9 # Restart configuration
SHELL
```

##### Slave
```bash
venus.vm.provision "shell", inline: <<-SHELL
  #Here put file provision
  cp -v /vagrant/config/resolution/resolv.conf /etc/ # Save DNS IPv4 resolution
  cp -v /vagrant/config/default/named /etc/default/ # Save IPv4 listening only enable
  cp -v /vagrant/config/venus/named.conf.options /etc/bind/ # Save DNS options 
  cp -v /vagrant/config/venus/named.conf.local /etc/bind/ # Save direct zone and inverted zone
  mkdir -p /etc/bind/zones # If exist make directory zones
  touch /etc/bind/zones/db.sistema.test # Create empty direct zone file  
  touch /etc/bind/zones/db.192.168.57 # Create empty direct zone file  
  sudo systemctl restart bind9 # Restart configuration
SHELL
```
## Author

**Lucas García Díaz**  
Github: [github.com/lgardia1](https://github.com/lgardia1)  
Email: [lgardia026@ieszaidinvergeles.org](lgardia026@ieszaidinvergeles.org)




