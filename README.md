# DNS-SISTEMA.TEST HomeWorks:

## 1. GitHub Repository
The first step of the practice involves creating a GitHub repository to manage all the files and configurations necessary for implementing a DNS server using **Vagrant** and network configurations for virtual machines. This repository will be used to version the configuration scripts, documentation, and tests.


## 2. Configure VMs

#### 2.1 Create vagrant file
```bash
vagrant init
```

#### 2.2 Edit Vagrantfile
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
In this configuration, we created two VMs-. The first **"earth"**, this will be the master server, and **"venus"** will be slave server. We also installed bind9 in both VMs.


#### 2.2 Start Vagrantfile
```bash
vagrant up
```

## 3. Configure server

### 3.1 Server Master (Earth)

We will SHH into the Master server (Earth)
```bash
vagrant ssh earth
```

##### 3.1.2 Protocol IPv4 and dnssec-validation

We go to the directory **`cd /etc/bind`**

###### DNS validaiton yes
We enable listening the server dnssec-validation yes.


```bash
sudo nano named.conf.options
```


This appears:
```bash
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        // forwarders {
        //      0.0.0.0;
        // 
        };
```


So we changed to this:
```bash
options {
        directory "/var/cache/bind";

        dnssec-validation yes;//security conf

        listen-on { any; }; //Listening in any IP
        listen-on-v6 { none; };//Isn't listening on IPv6
};
```

###### Check configuration:
```bash
sudo named-checkconf
```

If there isn't output text, it means everything is fine.

###### Only listen in IPv4

```bash
sudo nano /etc/default named
```

Change to this:
```bash
# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-u bind -4"
```
We are adding the param **-4** to listen only in IPv4

##### 3.1.2 ACL configuration

Edit this file
```bash
sudo nano named.conf
```


Appears this:
```bash
// This is the primary configuration file for the BIND DNS server named.
//
// Please read /usr/share/doc/bind9/README.Debian for information on the
// structure of BIND configuration files in Debian, *BEFORE* you customize
// this configuration file.
//
// If you are just adding zones, please do that in /etc/bind/named.conf.local

include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```





## 2. Problem Data

### 2.1. Network
The network used for this practice is **192.168.57.0/24**. All machines, both real and virtual, will be connected to this network.

### 2.2. Equipment
The following machines will be configured, some real and others fictitious:

| Equipment | FQDN                  | IP               | OS                    |
|-----------|-----------------------|------------------|-----------------------|
| Mercury   | mercurio.sistema.test  | 192.168.57.101   | Debian Linux graphical (ficticius)         |
| Venus     | venus.sistema.test     | 192.168.57.102   | Debian (text) |
| Earth     | tierra.sistema.test    | 192.168.57.103   | Debian (text) |
| Mars      | marte.sistema.test     | 192.168.57.104   | Windows graphical (fictitious) |

## 3. DNS Data

- The DNS server will only listen on IPv4.
- DNSSEC validation will be enabled (`dnssec-validation yes`).
- Recursive queries will only be allowed from machines within the networks **127.0.0.0/8** and **192.168.57.0/24**.
- The master server will be **tierra.sistema.test**, with authority over both forward and reverse zones.
- **Venus** will act as the slave DNS server, replicating configurations from **tierra.sistema.test**.
- Negative cache responses will last for two hours (in seconds).
- Unauthorized queries will be forwarded to the OpenDNS server **208.67.222.222**.
- The following aliases (CNAME) will be configured:
  - `ns1.sistema.test` will be an alias for **tierra.sistema.test**.
  - `ns2.sistema.test` will be an alias for **venus.sistema.test**.
  - `mail.sistema.test` will be an alias for **marte.sistema.test**.

## 4. Verification

Using tools like `dig` or `nslookup`, the following configurations will be verified:

- Correct resolution of **A** type records.
- Reverse resolution of IP addresses.
- Resolution of the aliases **ns1.sistema.test** and **ns2.sistema.test**.
- Query for the **NS** servers of the **sistema.test** zone.
- Query for the **MX** records of the **sistema.test** domain.
- Verification of zone transfer between the master and slave DNS servers (querying the **AXFR** record).
- Confirmation that both the master and slave servers can respond to the same queries.

## 5. Submission

You need to upload the following files to the Moodle platform:

1. The link to your GitHub repository.
2. A `.zip` file downloaded from the GitHub repository (this can be downloaded using the "Code" button and then selecting "Download ZIP").

## 6. Evaluation

The evaluation of the practice will be carried out as follows:

- 1 point for the GitHub infrastructure.
- 1 point for the Vagrant infrastructure.
- 2 points for the project documentation.
- 7 points for the DNS configuration, which will be verified by cloning the repository and running the tests using the `test.bat` (Windows) or `test.sh` (Linux) files.

## Author

**Lucas García Díaz**  
Github: [lgardia1](https://github.com/lgardia1)  
Email: [lgardia026@ieszaidinvergeles.org](lgardia026@ieszaidinvergeles.org)




