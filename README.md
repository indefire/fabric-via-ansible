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
1. generate a new pub/priv key set `$ ssh-keygen`
1. Copy the key to the target fabrc vm `$ ssh-copy-id ubuntu@${your_ip_here}`
You should be asked for the password to the vm that you created in the previous step
1. Logout and try to ssh back into the VM to verify you are not prompted for a password `ssh ubuntu@${target_ip_here}


## Sit back and let Ansible do the work
We need to modify ansible's hosts file so it knows which machine or machine we wish to perform the setup on. Then run the playbooks. The first playbook installs python via apt-get so when the second one runs, it can gather facts about the system.

- Modify ansible host file `sudo vim /etc/ansible/hosts`
- Setup the ip in this file. The below file contents is telling ansible the group's name is 'fabric' and the alias for the specific ip is fab1-4: 

```
[fabric]
fab1-4 ansible_host=${target_ip_here}
```
-. Verify the two playbook files have the targeted group defined `cat install-python.yaml` The -hosts: section should correspond to what we specified in the `[]` brackets in the hosts file 
```
- hosts:fabric
```
1. Run the ansible playbook to install python `$ ansible-playbook install-pythong.yaml`
1. Now run the playbook to install fabric, its dependencies, and nodejs for the examples `$ ansible-playbook fabric-1.4.yaml` You should see ok=19 changed=12 or something similar in the output with no failures

## Verify Fabric network
1. Start a new terminal (ansible added the ubuntu user to the docker group, so a new terminal is necessary)
1. `$ cd /home/ubuntu/fabric-samples/first-network`
1. Start fabric's example `$ ./byfn.sh up`

Now we have a functional network we can follow along with fabric's official tutorial and documentation here: [Fabric 1.4 build_network](https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html)
There are a good number of simple and complex examples within the fabric-samples directory, all of their descriptions are here: https://github.com/hyperledger/fabric-samples/ They are however constantly evolving.


## Useful information
- ssh-copy-id is not present, run this instead: `cat ~/.ssh/id_rsa.pub | ssh ubuntu@${your_ip_here} 'umask 0077; mkdir -p .ssh; cat >> .ssh/authorized_keys && echo "Key copied"'`   
- Fabric now supports GoLang, Java, and JavaScript chaincode. This is a huge improvement that Java and JavaScript is now supported in Fabric 1.4
- 
### Fabric Samples https://github.com/hyperledger/fabric-samples/
- /balance-transfer is the application we used as a basis for our application and heavily modified the nodejs implementation. I did not find the JWT use very helpful and more of a hinderence.
- /fabcar started out as its own separate application before they consolidated the samples, it is a very easy to understand, straight-forward nodejs sdk utilization
- /first-network was the basis for our fabric implementation and it makes creating your own network really easy by modifying their existing scripts
- /high-throughput is a more complex implementation of chaincode released much later than our initial endeavor. It shows off a lot of the later features such as pruning

### Fabric SDKs https://hyperledger-fabric.readthedocs.io/en/release-1.4/fabric-sdks.html 
There are a good number of SDKs all in various states of development. We have used the nodesdk and the JavaSDK. I found the nodesdk to be pretty easy to use, but as a Java developer I am drawn to the [JavaSDK](https://github.com/hyperledger/fabric-sdk-java).  The java sdk is pretty mature and we have used it on another project, however, I find the test cases pretty difficult to follow. I have not looked at it since the 1.2 release so it might have been streamlined since then.

### XRDP Install on an AWS instance
https://datawookie.netlify.com/blog/2017/08/remote-desktop-on-an-ubuntu-ec2-instance/
- Enable Password Login
```
sudo apt update
sudo apt install -y ubuntu-desktop xrdp
vim /etc/ssh/sshd_config
//search for PasswordAuthentication and change from no to yes
PasswordAuthentication yes
sudo passwd ubuntu
sudo systemctl restart ssh

```
- Before logging on via xrdp create a new file to disalbe known ubuntu issue with "Create Managed Color Device" popup as explained here: http://c-nergy.be/blog/?p=12043
```
sudo vim /etc/polkit-1/localauthority.conf.d/02-allow-colord.conf
```
Add the below contents:
```
polkit.addRule(function(action, subject) {
 if ((action.id == "org.freedesktop.color-manager.create-device" ||
 action.id == "org.freedesktop.color-manager.create-profile" ||
 action.id == "org.freedesktop.color-manager.delete-device" ||
 action.id == "org.freedesktop.color-manager.delete-profile" ||
 action.id == "org.freedesktop.color-manager.modify-device" ||
 action.id == "org.freedesktop.color-manager.modify-profile") &&
 subject.isInGroup("{users}")) {
 return polkit.Result.YES;
 }
 });
 
```
restart the vm
`sudo reboot`
