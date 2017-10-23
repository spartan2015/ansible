#Ansible - automation of server administration

command - simultanesly run on remote computer without logging in

playbook - collection of commands


##Install on windows

Need a linux machine for ansible control center
Create a VirtualBox and start it. Connect to it and install ansible"

Install VirtualBox: virtualbox.org
Install Vagrant: vagrantup.com


Create a Vagrant file - to create the virtual machines:

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

	config.vm.define "control" do |ctrl|
 	 ctrl.vm.box = "ubuntu/trusty64"
	 ctrl.vm.hostname = "ubuntu-control"
	 ctrl.vm.network "private_network", ip: "192.168.57.2"
         ctrl.vm.provider "virtualbox" do |vb|
		vb.memory = 1024			
	
		end

	end

	config.vm.define "web1" do |web01|
	 web01.vm.box = "jptoto/Windows2012R2"
	 web01.vm.hostname = "windows-webserver01"
  	 web01.vm.communicator = "winrm"
	 web01.winrm.username="vagrant"
	 web01.winrm.pasword="vagrant"
	 web01.vm.network "private_network", ip: "192.168.57.3"
	 web01.vm.provider "virtualbox" do |vb|
		vb.memory = 2048
		vb.cpus = 2
	 end		
	end
end


# back in the command line with the file exec:

vagrant up 


Connect to the connected vagrant virtual box:


#Test if WinRM is avaiable on a remote computer - with powershell:
Test-WSMan 192.168.57.3

#atlas.hashicorp.com - holds the vms - we can upload our own


#Now install ansible on ubuntu vm

#install python Then

ssh vagrant@192.168.57.2
sudo pip install markupssafe
sudo pip install xmltodict
sudo pip install piwinrm
sudo pip install ansible

ansible --version

mkdir pluralsight
cd pluralsight
nano inventory inventory.yml

---
[web]
192.168.57.3

save

mkdir group_vars
cd group_vars
nano web.yml

ansible_user: vagrant
ansible_password: vagrant
ansible_port: 5985
ansible_connection: winrm
ansible_winrm_cert_validation: ignore

cd ../
ansible web -i inventory.yml -m win_ping

#should be working correctly

#Using windows modules - to handle windows commands: add users, install, copy files, etc... (win_something)

# on ansibleview files
cd pluralsight
tree 
#gather info about the servers we control:
ansible web -i inventory.yml -m setup

nano ansible.cfg

[defaults]
inventory = ~/plurasight/inventory.yml

save
#Now you can just
ansible web -m setup
#execute dir
ansible web -m raw -a "dir"
ansible web -m raw -a "ipconfig"
ansible web -m win_service -a "name=spooler"
ansible web -m win_service -a "name=spooler state=stopped"
ansible web -m win_feature -a "name=Telnet-Client state=present"

#Using modules with playbooks - collection of commands and modules calls

E.g.:
playbook.yml

---
- hosts: web
 tasks:
 - name: Do a thing
  win_feature: "name=Package state=present"

 - name: Do another thing
  win_service: "name=W3SVC state=restarted"



# web server playbook
nano iis.yml

---
- hosts: web
 tasks: 
 - name: Endure II web server is installed
  win_feature: 
   name: Web Server 
   state: present
  when: ansible_os_family =="Windows"
 - name: Install start page 
  template: 
   src: iisstart.j2
   dest: c:\inetpub\wwwroot\iisstart.htm
 
nano iisstart.j2
<html>
<h1>Hello {{ansible_fqdn}}</h1>
</hmtl>


#Save

ansible-playbook iis.yml


# setting up Roles: compartimentalized collections o tasks, templates, varialbes and more: "web server", "email server", etc

#in the pluralsight folder:
cd ~/pluralsight
mkdir roles
mkdir roles/webserver
mkdir roles/webserver/tasks
mkdir roles/webserver/templates
nano roles/webserver/tasks/main.yml
---

 - name: Endure II web server is installed
   win_feature: 
    name: Web Server 
    state: present
   when: ansible_os_family =="Windows"
  - name: Install start page 
   template: 
    src: iisstart.j2
    dest: c:\inetpub\wwwroot\iisstart.htm
 
nano roles/webserver/templates/iisstart.j2
as above

nano ~/pluralsight/webservers.yml
---
- hosts: web
  roles: 
   - webserver 

ansible-playbook webservers.yml



