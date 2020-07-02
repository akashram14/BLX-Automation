# BLX-Automation
The Ansible scripts for automating blx installation and configuration

To run the script:
ansible-playbook blx.yml -k (-k prompts for the ssh password)

**I had some problems with by VM ,I updated centos and some dependencies were upgraded to another version so installation didn't work in my PC.**

# Finding interfaces:

The host node's ethernet interface is dynamically found out using shell commands and some regular expressions.

The host's default gateway is also found in a similar manner.

# Template file for blx.conf
The blx.conf file is saved in j2 format in the management system. The IP addresses and ha commands are given as variables. These variables are defined in the playbook/hosts file.


