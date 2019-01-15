# fabric-via-ansible
A short and quick install of fabric via ansible. The idea is to setup one master VM that will run ansible and then one or many machines that ansible will install Hyperledger Fabric on. First, Ansible will be installed on the master VM, then the other machine(s) will have additional ssh access created. The additional ssh access is only necessary if operating in a more constrained environment, a normal AWS environment does not need these extra steps.

### Install Ansible on dedicated vm
1. `$ sudo apt-get update`
1. `$ sudo apt-get install software-properties-common`
1. `$ sudo apt-add-repository --yes --update ppa:ansible/ansible`
1. `$ sudo apt-get install ansible`

### Create new ec2 instance(s) for target Fabric environment and prepare access
This step is only necessary if operating in a restrictive environment and additional identity keys need generated than the ones already present. 
  1. ssh to target vm
  1. edit the sshd_config `$ sudo vim /etc/ssh/sshd_config`
  1. search for 'PasswordAuthentication' and modify it from 'no' to 'yes' 
  1. save the change
  1. create a password for the user, in this case ubuntu `$ sudo passwd ubuntu`
  1. restart the ssh service `$ sudo service ssh restart`
  1. verify you can ssh to target machine with password authentication and accept ip as a known host


