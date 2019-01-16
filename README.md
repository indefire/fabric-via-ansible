# fabric-via-ansible
A short and quick install of fabric via ansible. The idea is to setup one master VM that will run ansible and then one or many machines that ansible will install Hyperledger Fabric on. First, Ansible will be installed on the master VM, then the other machine(s) will have additional ssh access created. The additional ssh access is only necessary if operating in a more constrained environment, a normal AWS environment does not need these extra steps.

## Install Ansible on dedicated vm
1. `$ sudo apt-get update`
1. `$ sudo apt-get install software-properties-common`
1. `$ sudo apt-add-repository --yes --update ppa:ansible/ansible`
1. `$ sudo apt-get install ansible`

## Create new ec2 instance(s) for target Fabric environment and prepare access

#### Enable password authentication on target vm
This step is only necessary if operating in a restrictive environment and additional identity keys need generated than the ones already present. 
  1. ssh to target vm
  1. edit the sshd_config `$ sudo vim /etc/ssh/sshd_config`
  1. search for 'PasswordAuthentication' and modify it from 'no' to 'yes' 
  1. save the change
  1. create a password for the user, in this case ubuntu `$ sudo passwd ubuntu`
  1. restart the ssh service `$ sudo service ssh restart`
  1. verify you can ssh to target machine with password authentication and accept ip as a known host

### Create new identity key from Ansible VM
1. generate a new pub/priv key set
`$ ssh-keygen`
1. Copy the key to the target fabrc vm `$ ssh-copy-id ubuntu@${your_ip_here}`
You should be asked for the password to the vm that you created in the previous step
1. Logout and try to ssh back into the VM to verify you are not prompted for a password `ssh ubuntu@${target_ip_here}


## Sit back and let Ansible do the work
We need to modify ansible's hosts file so it knows which machine or machine we wish to perform the setup on. Then run the playbooks. The first playbook installs python via apt-get so when the second one runs, it can gather facts about the system.

1. Modify ansible host file `sudo vim /etc/ansible/hosts`
1. Setup the ip in this file. The below file contents is telling ansible the group's name is 'fabric' and the alias for the specific ip is fab1-4: 
```
[fabric]
fab1-4 ansible_host=${target_ip_here}
```
1. Verify the two playbook files have the targeted group defined `cat install-python.yaml` The -hosts: section should correspond to what we specified in the [] brackets in the hosts file
```
- hosts:fabric
```
1. Run the ansible playbook to install python `$ ansible-playbook install-pythong.yaml`
2. Now run the playbook to install fabric, its dependencies, and nodejs for the examples `$ ansible-playbook fabric-1.4.yaml`


## Useful information
1. ssh-copy-id is not present, run this instead: `cat ~/.ssh/id_rsa.pub | ssh ubuntu@${your_ip_here} 'umask 0077; mkdir -p .ssh; cat >> .ssh/authorized_keys && echo "Key copied"'`   
