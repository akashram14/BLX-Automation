# BLX-Automation
The Ansible scripts for automating blx installation and configuration

# Table of contents
1. [Installation](#Installation)
2. [Extracting tar file](#Extracting-tar-file)
3. [Installing BLX](#Installing-blx)
4. [Start BLX](#start-blx)
5. [Start BLX in dedicated mode](#start-blx-dedicated)

# Installation

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
# Installing BLX
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
# Start BLX in dedicated mode
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

# Setting up a HA pair
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


# Finding interfaces:

The host node's ethernet interface is dynamically found out using shell commands and some regular expressions.

The host's default gateway is also found in a similar manner.

# Template file for blx.conf
The blx.conf file is saved in j2 format in the management system. The IP addresses and ha commands are given as variables. These variables are defined in the playbook/hosts file.


To run the script:
ansible-playbook blx.yml -k (-k prompts for the ssh password)

**I had some problems with by VM ,I updated centos and some dependencies were upgraded to another version so installation didn't work in my PC.**



