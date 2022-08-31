# **SmitKub**
To document and deploy an ansible backed Kubernetes cluster. 

- [**SmitKub**](#smitkub)
- [**Whats it for**](#whats-it-for)
- [**Components**](#components)
- [**Home Network Setup**](#home-network-setup)
- [**Ansible Setup**](#ansible-setup)
- [ScratchNotes](#scratchnotes)


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


# ScratchNotes
