# DNS-SISTEMA.TEST HomeWorks

## Index
- [DNS-SISTEMA.TEST HomeWorks](#dns-sistematest-homeworks)
  - [Index](#index)
    - [Introduction](#introduction)
  - [1. GitHub Repository](#1-github-repository)
  - [2. Configure VMs](#2-configure-vms)
    - [2.1 Create Vagrantfile](#21-create-vagrantfile)
    - [2.2 Edit Vagrantfile](#22-edit-vagrantfile)
    - [2.3 Start Vagrantfile](#23-start-vagrantfile)
  - [3. Configure Servers](#3-configure-servers)
    - [3.1 Master Server (Earth)](#31-master-server-earth)
      - [3.1.1 Listen only on IPv4](#311-listen-only-on-ipv4)
      - [3.1.2 Enable dnssec-validation and IPv4](#312-enable-dnssec-validation-and-ipv4)
      - [3.1.3 ACL Configuration](#313-acl-configuration)
      - [3.1.4 Configure DNS Zone](#314-configure-dns-zone)
      - [3.1.5 Create Zone Database with alias and mail server](#315-create-zone-database-with-alias-and-mail-server)
        - [What does "The server marte.sistema.test will act as the mail server for the domain sistema.test" mean?](#what-does-the-server-martesistematest-will-act-as-the-mail-server-for-the-domain-sistematest-mean)
  - [Author](#author)


### Introduction

This documentation gives you the basic setup and configurations needed for setting up DNS servers using Bind9 with Vagrant, along with the proper structure for GitHub repositories and DNS configurations. Feel free to expand further as needed.



---

## 1. GitHub Repository
The first step of the practice involves creating a GitHub repository to manage all the files and configurations necessary for implementing a DNS server using **Vagrant** and network configurations for virtual machines. This repository will be used to version the configuration scripts, documentation, and tests.

---

## 2. Configure VMs

### 2.1 Create Vagrantfile
To start the configuration, you need to create a `Vagrantfile` using the following command:

```bash
vagrant init
```

### 2.2 Edit Vagrantfile
The **Vagrantfile** should be edited to create two VMs: one for the master DNS server (Earth) and the other for the slave DNS server (Venus). The configuration below installs the required packages like **bind9** on both machines.

```ruby
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
```

### 2.3 Start Vagrantfile
To start the virtual machines, use the following command:
```bash
    vagrant up
```
---

## 3. Configure Servers

### 3.1 Master Server (Earth)

#### 3.1.1 Listen only on IPv4

SSH into the Master Server (Earth) and modify the following file:
```bash
    vagrant ssh earth
    sudo nano /etc/default/named
```

Modify the startup options to only listen on IPv4 by adding **-4**:
```bash
OPTIONS="-u bind -4"
```

#### 3.1.2 Enable dnssec-validation and IPv4
Edit the DNS options in the **named.conf.options** file to enable DNSSEC validation and IPv4-only listening.
```bash
    sudo nano /etc/bind/named.conf.options
```

Modify it as follows:
```bash
    options {
            directory "/var/cache/bind";
            dnssec-validation yes;
            listen-on { any; }; // Listening on IPv4 only
            listen-on-v6 { none; }; // Disabling IPv6 listening
    };
```

Check the configuration:
```bash
sudo named-checkconf
```

#### 3.1.3 ACL Configuration
To limit recursive queries to specific networks, configure ACLs in the same **named.conf.options** file:
```bash
    acl "redes_permitidas" {
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

#### 3.1.4 Configure DNS Zone
```bash
Next, we define the DNS zone for tierra.sistema.test in the **named.conf.local** file:
sudo nano /etc/bind/named.conf.local
```

Add the following zone configuration:
```bash
    zone "tierra.sistema.test" {
            type master;
            file "/etc/bind/zones/db.sistema.test";
};
```

Create the directory for zone files:
```bash
    sudo mkdir /etc/bind/zones
```

Check the configuration again:
```bash
    sudo named-checkconf
```

#### 3.1.5 Create Zone Database with alias and mail server


**Edit the DNS zone file (`db.sistema.test`)**:
```bash
    sudo nano /etc/bind/zones/db.sistema.test
```
Finally, create the zone database file for tierra.sistema.test:
```bash
    sudo nano zones/db.sistema.test
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
@       IN      MX 10   marte.sistema.test.
```
##### What does "The server marte.sistema.test will act as the mail server for the domain sistema.test" mean?

This statement means that the server named `marte.sistema.test` will handle email processing for the domain **sistema.test**. In other words, any emails sent to addresses like `user@sistema.test` will be managed and processed by the server `marte.sistema.test`.

To configure this correctly, you need to add an **MX (Mail Exchange) record** to your DNS zone file. An MX record specifies the mail server responsible for receiving emails for a particular domain.

## Author

**Lucas García Díaz**  
Github: [github.com/lgardia1](https://github.com/lgardia1)  
Email: [lgardia026@ieszaidinvergeles.org](lgardia026@ieszaidinvergeles.org)




