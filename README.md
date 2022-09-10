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
  - [kubernetes installation methods](#kubernetes-installation-methods)
    - [organization](#organization)
      - [Understand Ansible Roles and galaxy better.](#understand-ansible-roles-and-galaxy-better)
        - [roles](#roles)
        - [galaxy](#galaxy)
      - [manage the inventory file](#manage-the-inventory-file)
    - [manage secrets](#manage-secrets)
      - [ansible vault](#ansible-vault)
- [Victory](#victory)


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

## kubernetes installation methods


* [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
* [kubespray](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/)
* [kubespray/equinix metal](https://kubespray.io/#/docs/equinix-metal)
  
  leaning towards kubeadm as its the "supported" release.


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


this seems to work with the sample playbook below. running additonal apt updates and upgrades did not upgrade vault.
```
---
- name: apt lock testing
  hosts:
  - smitkub1.smithsonite.home
  become: true

  tasks:
  - name: Run the equivalent of "apt-get update" as a separate step
    ansible.builtin.apt:
      update_cache: yes

  - name: install hashicorp vault
    ansible.builtin.apt:
      name: vault=1.11.2-1
      state: present

  - name: Prevent vault from being upgraded
    ansible.builtin.dpkg_selections:
      name: vault
      selection: hold
```

Currently on pre-flight

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/

```
[init] Using Kubernetes version: v1.24.0
[preflight] Running pre-flight checks
        [WARNING SystemVerification]: missing optional cgroups: blkio
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR CRI]: container runtime is not running: output: time="2022-09-07T21:18:17-04:00" level=fatal msg="unable to determine runtime API version: rpc error: code = Unavailable desc = connection error: desc = \"transport: Error while dialing dial unix /var/run/containerd/containerd.sock: connect: no such file or directory\""
, error: exit status 1
        [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
        [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
        [ERROR KubeletVersion]: the kubelet version is higher than the control plane version. This is not a supported version skew and may lead to a malfunctional cluster. Kubelet version: "1.25.0" Control plane version: "1.24.0"
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```


kubelet versions 
* 1.25.0-00
* 1.24.4-00
* 1.24.3-00
* 1.24.2-00

kubectl version
* 1.25.0-00
* 1.24.4-00
* 1.24.3-00
* 1.24.2-00

kubeadm version
* 1.25.0


ended the day here
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd

resource?
https://computingforgeeks.com/install-kubernetes-cluster-ubuntu-jammy/

# Victory
![Screenshot](Docs/pics/victory.png)
```
ansible@autobot:~$ sudo kubeadm init --config kubeadm-config.yaml
[init] Using Kubernetes version: v1.25.0
[preflight] Running pre-flight checks
        [WARNING SystemVerification]: missing optional cgroups: blkio
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [autobot kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.1.230]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [autobot localhost] and IPs [192.168.1.230 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [autobot localhost] and IPs [192.168.1.230 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[apiclient] All control plane components are healthy after 46.506945 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node autobot as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]    
[mark-control-plane] Marking the node autobot as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: tyyxjh.6p6046i6os50ovbz
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.230:6443 --token tyyxjh.6p6046i6os50ovbz \
        --discovery-token-ca-cert-hash sha256:aaf0b576c83cc2f4d6714cf400671a50a308e91633f26445fe94c8b96fe9deea
```


much of the configuration can be found in /etc/kubernetes/ now.
The kubeadm init command and its config are my consern now. There are commands one passes in to set specific configs such as the cidr range etc. id like all of that in a config file... as is shown when configuring the [cgroup driver](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/). id like to see where those can go into the specified config file.

modifying the pod cidr 
https://tufora.com/tutorials/kubernetes/change-kubernetes-pods-cidr

install flannel
```
kubectl apply -f kube-flannel.yml
```

Going to look into a kubernetes class on puralsight now.