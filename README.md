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

# Finding interfaces:

The host node's ethernet interface is dynamically found out using shell commands and some regular expressions.

The host's default gateway is also found in a similar manner.

# Template file for blx.conf
The blx.conf file is saved in j2 format in the management system. The IP addresses and ha commands are given as variables. These variables are defined in the playbook/hosts file.


To run the script:
ansible-playbook blx.yml -k (-k prompts for the ssh password)

**I had some problems with by VM ,I updated centos and some dependencies were upgraded to another version so installation didn't work in my PC.**



