# **SmitKub**
To document and deploy an ansible backed Kubernetes cluster. 

- [**SmitKub**](#smitkub)
- [**Whats it for**](#whats-it-for)
- [**Components**](#components)
- [**Home Network Setup**](#home-network-setup)
- [**Ansible Setup**](#ansible-setup)
  - [**RPI Setup**](#rpi-setup)
    - [**Control system "Autobot"**](#control-system-autobot)
    - [**Worker Systems**](#worker-systems)
  - [**System maint Playbooks**](#system-maint-playbooks)
    - [**Cron job**](#cron-job)
- [ScratchNotes](#scratchnotes)
  - [todo](#todo)
    - [organization](#organization)
      - [Understand Ansible Roles and galaxy better.](#understand-ansible-roles-and-galaxy-better)
        - [roles](#roles)
        - [galaxy](#galaxy)
      - [manage the inventory file](#manage-the-inventory-file)
    - [manage secrets](#manage-secrets)
      - [ansible vault](#ansible-vault)


# **Whats it for**
The environment is used for several things. Firstly it is used in order to provide a kubernets lab environment. Second it is used as a method of hosting some home utilities such as a plex server, and potentially other services such as pihole. 

# **Components**
This cluster is designed around haveing 4 raspberry pi 4b 8gb units. One control module and 3 workers. The control module will perform triple duty as an ansible controller and GitHub/ADO runner.

# **Home Network Setup**
The nodes have DHCP reservations for the "smithsonite.home" network. Their information is as follows

<style>
table {
    border-collapse: collapse;
}
table, th, td {
   border: 1px solid black;
}
blockquote {
    border-left: solid blue;
	padding-left: 10px;
}
</style>


| Name | Mac | IP |
| :---: | :---: | :---: |
| Autobot | E4:5F:01:B9:98:00 |192.168.1.230 |
| smitkub1 | DC:A6:32:C7:00:71 | 192.168.1.232 |
| smitkub2 | DC:A6:32:C1:69:C2 | 192.168.1.233 |
| smitkub3 | E4:5F:01:6F:6D:33 | 192.168.1.234 |

# **Ansible Setup**
The control pane is autobot.smithsonite.home. From this system we can control the other 3. 
Ansible is installed and running under a user named "ansible". It has an SSH keypair (found under ansible SSH in keeper). This keypair is to be uploaded to the RPI's for the ansible users.

## **RPI Setup**
The pi's are configured with ubuntu 22.04 and with my own SSH keypair with the user csmithson and the campsmit network.
Additional users will be configured as needed. The first user should be "ansible".

### **Control system "Autobot"**

 ```
 sudo useradd -d /home/ansible -m -s/bin/bash ansible
 sudo mkdir /home/ansible/.ssh
At this time, copy the id_rsa and id_rsa.pub files from keeper into the /home/ansible/.ssh directory.
 sudo chmod 600 /home/ansible/.ssh/id_rsa
 sudo chmod 644 /home/ansible/.ssh/id_rsa.pub
 sudo chown ansible:ansible /home/ansible/.ssh -R
 echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
 sudo apt install python3-pip
 ```
The user may be assumed locally by executing
```
sudo su
su ansible
```


### **Worker Systems**

 ```
 sudo useradd -d /home/ansible -m -s/bin/bash ansible
 sudo mkdir /home/ansible/.ssh
 echo ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC+U3naODIjFl07dJ9YzYUEZx7yIITCLMRy2ijEWkrrxRZgkaq/ZfAV9KaHQoCEzDtITosXIy4yDbgk4dYJw+1tDfbB3U3VUde2SaYUD3YWHyCezHhlafKdjacXZYS9fdLq0iagPs0+Rs7ORpLaiKhH78XPG4tN6ead4dG/7roTCNG67pqD7yWUL/AYe4qVMNwGNwiBG+0+CQKH2FKkhXKhDLY1r+vizf7mkehczbaF75EJbe/FxPIUoxs8GL5CFOJIsK71KW9AHuQAXmBec1iVvk9GcV4UX26ejpkOQvWRKUjTK0uXdAF1jsKQLHKbFUhSAi51H3ZZShv5v50oQAkWEULxjZ5CYfZIpsVqOZOldcxW4kYPg+L93ArvMwNhV7kKxuL88kG5Sp24QlrJ7l2L5THT2IfmANz2uF8c7HuUTwy+10iN1x+wZpeTqVktvP9N8DokrdqT20q0VqLuo1oAZloeyCJGvEj0LLMr3paSTiNs5z7GF8+PfzqO7GhhZtoWqSn9VmNHLPoAYox8lyyi1WpCyt/bQh5iIwWoC4pWWyn57RHhyGiaLfoFBOcLvSFwgXyV6CJr8g7f2kKgowyzxteqDieVR8c9XlKJdVgvA+tJlGjQbP2+zgquazqOv+4wHwJKh1tjTzrCqHv27/lNHr2ngTXDbuHuTfFv9Y4Zrw== ansible@autobot | sudo tee /home/ansible/.ssh/authorized_keys
 sudo chmod 600 /home/ansible/.ssh/authorized_keys
 sudo chown ansible:ansible /home/ansible/.ssh -R
 echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
 sudo apt install python3-pip -y
 ```
The user may be assumed locally by executing
```
sudo su
su ansible
```

## **System maint Playbooks**
The following playbooks were created in order to handle the following  purposes
* [picool](playbooks/picool/main.yml) - This is the fan software for the classic pi cases
  * This has a dependency of the [picool](https://galaxy.ansible.com/csmithson12345/picool) ansible role
* [updates](playbooks/updates/main.yml) - This performs apt updates and upgrades
  * This has a dependency of the [apupdate](https://galaxy.ansible.com/csmithson12345/aptupdate) ansible role

### **Cron job**
while some cron jobs will apparently leverage another user... using crontab -e dosent actually seem to do so... nor does it seem to obey any path or home modification.
what i was able to do was execute a command as the ansible user leveraging su. it STILL does not leverage the path that the ansible user has... so

* you need to navigate to the ansible dir for the specific playbook because you cannot pump the ansible.cfg in any other way
* you need to use the FULL path to ansible-playbook in order to invoke it

as sudo 
```
crontab -e
```

```
5 1 * * * su ansible -c "cd /home/ansible/git/SmitKub/playbooks/updates && /home/ansible/.local/bin/ansible-playbook main.yml"
```


# ScratchNotes
## todo
Settle on a method to schedule jobs ( a cronjob direct on autobot, or a github-actions based schedule)



### organization
In the same way i handled the primary pipeline and child piplines... you can use the import_tasks to bring it other files containing tasks.



#### Understand Ansible Roles and galaxy better.
##### roles
make a roles folder
make a folder named for the role
2 things are required
a meta directory
a tasks folder

roles are automatically picked up from the global roles folder /etc/ansible/roles.
they are also automatically picked up if there is a roles folder in  the DIR your exeucting from
youc an also configure the ansible.cfg file to point to a roles dir.

typically good practice to make an ansible.cfg file in your project directory
inside of that you will want to specify a roles path *(usually ./roles)
that way conflicting versions of a role will not overwrite eachother.


##### galaxy
The ansible-galaxy role init  will create a "scaffold" of a proper role
```
ansible-galaxy role init test
```



#### manage the inventory file

### manage secrets
#### ansible vault
ansible vault allows you to encrypt fiels and store them in source control
i am not a fan  of that.
it alos seems to need a password to access it. 

hold back a specific apt package with apt hold
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/dpkg_selections_module.html

find avaliable versions
```
apt-cache showpkg <package-name>
```
install a specific version
```
sudo apt install package=version -V
```
sudo apt install vault=1.11.2-1 -V

