# BLX-Automation
The Ansible scripts for automating blx installation and configuration

# Table of contents
1. [Installation](#Installation)
2. [Extracting tar file](#Extracting-tar-file)
3. [Installing BLX](#Installing-blx)
4. [Start BLX](#start-blx)
5. [Start BLX in dedicated mode](#start-blx-dedicated)

## Installation

BLX can be installed by downloading the tar file from [Citrix BLX packages](https://www.citrix.com/downloads/citrix-adc/bare-metal-adc/?_ga=2.262364228.913521931.1596445047-385435695.1581332042). Previous releases can also be downloaded from the same page.

# <div id='Extracting-tar-file'/>
# Extracting tar file 
The tar file can be extracted by typing the following command in the folder in which the tar file is downloaded.
```bash
tar -xvf blx-<release number>-<build number>.tgz
```
For blx release number 13.0 and build number 41.20, the following command should be executed.
```bash
tar -xvf blx-13.0-41.20.tgz
```
This extracts the necessary files in the folder with name 
```bash
blx-<release number>-<build number>
```

# <div id='Installing-blx'/>
## Installing BLX
Rub the following command in the directory of BLX.

Syntax:
```bash
cd blx-<release number>-<build number>
yum install blx*.rpm
```

Example:
```bash
cd blx-13.0-41.20
yum install blx*.rpm
```

# <div id='start-blx'/>
# Start BLX

```bash
systemctl start blx
```
This starts BLX in shared mode.

```bash
systemctl status blx
```
This command gives the status of blx service. That is, whether it is active or not. 

# <div id='start-blx-dedicated'/>
## Start BLX in dedicated mode
To start BLX in dedicated mode, we have to provide an IP address dedicated for BLX.

The free IP address can be obtained by running the command.

```bash
nmap -sn <network_name>/<bitmask>
nmap -sn 172.16.18.55/24
```
This command lists the hosts which are active. We have to choose any of the free addresses available. 

The free IP address should be specified in the IP address field of blx.conf file.

The default path to blx.conf is 
```bash
/etc/blx/blx.conf
```

Along with specifying the free ip in the blx.conf file we need to configure a dedicated interface or a sub interface for the blx to start in dedicated mode.
The number of worker processes must be uncommented.

Now we need to stop and again start blx.
```bash
systemctl stop blx
systemctl start blx
```
Now the blx will start in dedicated mode and this can be reconfirmed by typing the following command in the command prompt:-
```bash
cli_script.sh "show_ip"
```
This prints out a table showing the netscalar ip as the ip address that we has given in the blx.conf file.

Now we have blx up and running in dedicated mode.
So now we need to set up the HA functionality in blx.

## Setting up a HA pair
Now we need to set up HA in the two systems having blx configured in dedicated mode.
 The IPs of each system are given to each other. The command used for this step is:

```bash
cli_script.sh “add ha node <id> <other-system-ip> 
Eg: cli_script.sh “add ha node 1 172.168.18.25”
```
If the systems are located in different subnets, then the argument ‘inc’ should be enabled.

Eg: The first systems IP is 172.16.18.50 and second systems IP is 10.0.0.15. Then HA node can be added from IP 172.168.18.50 to IP 10.0.0.15 by using the following command.

```bash
cli_script.sh “add ha node 1 172.168.18.25 -inc ENABLED”
```
Now the HA pair has been set with one as primary and the other as secondary.

Now the HA pair needs to have failSafe mode set which is done by the following command:-

```bash
set HA node -failSafe ON
```

Now we need to enable the load balancing functionality.
This is done by adding a virtual load balancing server.Which is achieved by running the nmap command.

```bash
nmap -sn <network_name>/<bitmask>
nmap -sn 172.16.18.55/24
```
This gives the list of IPs used by the hosts in the network. 
Any of the free IP addresses can be used for the virtual load balancing server. 
The command to add the load balancing server is:

```bash
cli_script.sh “add lb vserver  <lb-server-name>  <type>  <lb-vserver-ip> <port-number>”
Eg: cli_script.sh “add lb vserver  v1 HTTP  127.0.0.1 80”
```

The blx receives and sends its requests through a server. The server’s IP can be given as 192.0.0.3. This is the IP of the server at which BLX listens to.

The ADC command to add the services is:
```bash
cli_script.sh “add service <service-name> <service-ip> <type> <port-number>”
Eg: cli_script.sh “add service s1 192.0.0.3 HTTP 81”
```

The command to bind the service to vserver is:

```bash
cli_script.sh “bind lb vserver <lb-vserver-name> <service-name>”
Eg: cli_script.sh “bind lb vserver v1 s1”
```

This way multiple services can be connected to the network through the virtual load balancing server.

## Automation

To install and configure BLX in dedicated mode, we have
tested and developed an Ansible playbook. Ansible is a
popular dev-ops tool that allows one to write system
definitions in a YAML (abbreviation of "YAML Ain't Markup
Language"), and generate reproducible, scalable network
configurations.


### Setting up the management node

The network configuration to run the playbook requires the
following: 

  - Management Node: the playbook will be run from this node
  - Host nodes: the nodes to install BLX on and configure
High Availability

### System Requirements (Management Node)

Following are the requirements for the management node. These dependencies may be installed on the management node via a package manager such as yum.

-  Linux based OS (Cent OS, Fedora, RHEL)
-  Python3
-  Ansible set of tools
   * ansible
   * ansible-config
   * ansible-connection
   * ansible-console
   * ansible-doc
   * ansible-playbook
-  File containing host configuration
-  File containing service list

### System Requirements (Host Nodes)

Following are the requirements for the host node(s). 

- BLX compatible OS (systemd based init: CentOS, Fedora, RHEL etc.)
- Open to SSH connections
- Services running mentioned in the service list file.

### Tasks performed by the playbook

1. Fetch BLX on remote system(s)
The Ansible playbook downloads the latest version BLX onto the remote systems parallely and extracts the files, in preparation for installation.

2. Add nsroot user
Next, the playbook adds a user called “nsroot” and gives it the necessary permissions to run on the remote system. This is the user for logging into the remote system once BLX is up and running.

3. Install BLX
This task installs BLX from the extracted rpm files from the first task. Necessary dependencies are downloaded as required and updated to the latest version. If this task fails (insufficient permissions or invalid root user setup), the playbook exits preemptively.

4. Interface configuration
The main interface and default gateway for the given system are discovered. This task fetches the interface tagged as default by route, secondary interfaces if any, are not selected. The final installation of BLX will occupy this interface and any traffic through this interface are routed through BLX and therefore, the HA setup.

5. BLX Configuration
BLX is configured to run in HA mode. The input settings and services are applied and written to /etc/blx.conf. You may edit this template once the playbook finishes executing, based on requirements.

6. Start BLX
Finally, BLX is started in dedicated mode. Once BLX is running, you may no longer ssh into the default user and IP. The set of host nodes are now accessible via nsroot@<nsip>.

### Steps to run the playbook

1. Access the management node
Login into the management node. Navigate to the directory containing the playbook, hosts file, and (optionally) the service file. Your directory structure should look something like this:

```
# tree ./
.
├── services
├── hosts 
└── playbook.yml 

0 directories, 3 files 
```
	
2. Run the playbook
Run the playbook with the ansible-playbook command. You may pass the SSH password or key with the --ask-vault-pass or the --key-file flags respectively. The hosts file must be supplied to ansible via the -i flag, for example:

```
# ansible-playbook playbook.yml -i hosts --ask-vault-pass
```

3. Supply input (optional)

You may pass the services list in one of two ways:
  - Standard input (via prompt)
  - Source from a file

See Setting up Services for details on the service file format.

## Setting up Hosts

The hosts file should contain a simple description of the
hosts you wish to configure. Here is a sample hosts file:

```
# primary node in HA setup
[primary] 
root@192.168.1.7    # user@ip for remote host

[primary:vars]
nsip1=192.168.1.20  # NSIP of the BLX appliance once installed
haip1=192.168.1.21  # IP of backup system
port1=9080          # HTTP port to serve web interface
port2=9443          # HTTPS port to serve web interface
id=1                # Node ID in HA cluster

# secondary node in HA setup
[secondary]
root@192.168.1.8

[secondary:vars]
nsip1=192.168.1.21
haip1=192.168.1.22
port1=9080
port2=9443
id=2
```

In the above example, the haip of each node is assigned to
the nsip of the other node. This configuration is mandatory
to set up an HA Pair.

## Setting up Services

The services file should contain a list of services you have
on host machines, that BLX should know about and perform HA
over. It is a simple text file containing one service per
line, comments may begin with the # character.


Every line of the services file should be of the format:

```
name_of_service;IP_addr;type_of_service;port
```

1. `name_of_service` : Name for the service. Must begin with an
ASCII alphabetic or underscore (_) character, and must
contain only ASCII alphanumeric, underscore, hash (#),
period (.), space, colon (:), at (@), equals (=), and hyphen
(-) characters. Cannot be changed after the service has been
created


2. `IP_addr`: IP to assign to the service.


3. `serviceType`: 
Protocol in which data is exchanged with the service
Possible values: HTTP, FTP, TCP, UDP, SSL, SSL_BRIDGE, SSL_TCP, DTLS, NNTP, RPCSVR, DNS, ADNS, SNMP, RTSP, DHCPRA, ANY, SIP_UDP, SIP_TCP, SIP_SSL, DNS_TCP, ADNS_TCP, MYSQL, MSSQL, ORACLE, MONGO, MONGO_TLS, RADIUS, RADIUSListener, RDP, DIAMETER, SSL_DIAMETER, TFTP, SMPP, PPTP, GRE, SYSLOGTCP, SYSLOGUDP, FIX, SSL_FIX, USER_TCP, USER_SSL_TCP, QUIC, IPFIX, LOGSTREAM
	

4. `Port`: Port number of the service.
