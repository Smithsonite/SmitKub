# **SmitKub**
# **Purpose**
To document and deploy an ansible backed Kubernetes cluster. 

# **ToC**
- [**SmitKub**](#smitkub)
- [**Purpose**](#purpose)
- [**ToC**](#toc)
- [**Whats it for**](#whats-it-for)
- [**Components**](#components)
- [**Home Network Setup**](#home-network-setup)
- [**Ansible Setup**](#ansible-setup)
  - [**RPI Setup**](#rpi-setup)
    - [**Control system "Autobot"**](#control-system-autobot)
    - [**Worker Systems**](#worker-systems)
  - [**System maint Playbooks**](#system-maint-playbooks)
    - [**Cron job**](#cron-job)
- [**Kubernetes Installation**](#kubernetes-installation)
  - [**Node Prep**](#node-prep)
  - [**Kubernetes Cluster configuration**](#kubernetes-cluster-configuration)
    - [**Quick reference to recreating the cluster**](#quick-reference-to-recreating-the-cluster)
  - [**Configuration tools and methods**](#configuration-tools-and-methods)
    - [**Building the cluster**](#building-the-cluster)
      - [**Software packages**](#software-packages)
      - [**Bootstraping**](#bootstraping)
        - [**kubeadm created kubeconfig files**](#kubeadm-created-kubeconfig-files)
          - [**CA**](#ca)
          - [**Staic pod manifests**](#staic-pod-manifests)
      - [**Createing a control plane node**](#createing-a-control-plane-node)
      - [**Adding a node to a cluster**](#adding-a-node-to-a-cluster)
      - [**Add node to cluster**](#add-node-to-cluster)
  - [networking](#networking)
    - [ports](#ports)
  - [scalability](#scalability)
  - [high availability](#high-availability)
  - [Disaster Recovery](#disaster-recovery)
  - [Working with the Kubernetes cluster](#working-with-the-kubernetes-cluster)
    - [using kubectl to interact with the cluster](#using-kubectl-to-interact-with-the-cluster)
      - [opperations](#opperations)
      - [resources](#resources)
      - [output](#output)
      - [kubectl](#kubectl)
      - [demo](#demo)
    - [application deployments INTO the cluster](#application-deployments-into-the-cluster)
      - [declaritive](#declaritive)
        - [our manifest (manual review)](#our-manifest-manual-review)
        - [generating a manifest with a dry run (automatic and great)](#generating-a-manifest-with-a-dry-run-automatic-and-great)
        - [expose application](#expose-application)
        - [declaritive deployment](#declaritive-deployment)
- [9-26-2022](#9-26-2022)
- [successfull exposure](#successfull-exposure)
  - [confirmed](#confirmed)
    - [netplan config](#netplan-config)
  - [cleanup](#cleanup)
- [calico and external IP](#calico-and-external-ip)
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
- [**Resources**](#resources-1)


# **Whats it for**
The environment is used for several things. Firstly it is used in order to provide a kubernets lab environment. Second it is used as a method of hosting some home utilities such as a plex server, and potentially other services such as pihole. 

# **Components**
This cluster is designed around haveing 4 raspberry pi 4b 8gb units. One control module and 3 workers. The control module will perform triple duty as an ansible controller and GitHub/ADO runner.

# **Home Network Setup**
The nodes have DHCP reservations for the "smithsonite.home" network. Their information is as follows


| Name | Mac | Primary IP | Alternet IP |
| :---: | :---: | :---: | :---: |
| Autobot | E4:5F:01:B9:98:00 |192.168.1.230 | 192.168.2.235 |
| smitkub1 | DC:A6:32:C7:00:71 | 192.168.1.232 | |
| smitkub2 | DC:A6:32:C1:69:C2 | 192.168.1.233 | |
| smitkub3 | E4:5F:01:6F:6D:33 | 192.168.1.234 | |

# **Ansible Setup**
The control plane is autobot.smithsonite.home. From this system we can control the other 3. 
Ansible is installed and running under a user named "ansible". It has an SSH keypair (found under "ansible SSH" in keeper). This keypair is to be uploaded to the RPI's for the ansible users.

## **RPI Setup**
The pi's are configured with ubuntu 22.04 and with my own SSH keypair with the user csmithson and the campsmit network.
Additional users will be configured as needed. The first user should be "ansible". Once this user is configured, additional configuration can be done via Ansible.

### **Control system "Autobot"**

 ```
 sudo useradd -d /home/ansible -m -s/bin/bash ansible
 sudo mkdir /home/ansible/.ssh
 # At this time, copy the id_rsa and id_rsa.pub files from keeper into the /home/ansible/.ssh directory.
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
# **Kubernetes Installation**
## **Node Prep**
An ansible role has been created to deploy a consistent kubernetes installation for both worker and control nodes. The two playbooks for installation can be found in the playbooks directory
* [kubworkerinstall](playbooks/kubworkerinstall/main.yml)
* [kubeconrolinstall](playbooks/kubecontrolinstall/main.yml)

This will install all of the required packages. As of now, this will leverage version 1.25.


## **Kubernetes Cluster configuration**

### **Quick reference to recreating the cluster**
```
kubeadm config print init-defaults | tee ClusterConfiguration.yaml
sudo  kubeadm init --config=ClusterConfiguration.yaml --cri-socket /run/containerd/containerd.sock
mkdir -p $HOME/.kube
sudo cp - i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f calico.yaml
```
[ClusterConfiguration.yaml](bootstrap/ClusterConfiguration.yaml) and [calico.yaml](bootstrap/calico.yaml) can be found in the bootstrap directory.



[class link](https://app.pluralsight.com/courses/9f2f79a1-8408-4c5a-8060-e424161dc54e/table-of-contents)

## **Configuration tools and methods**
The standard control plane/cluster tool is [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

[Kubernetes](https://github.com/kubernetes/kubernetes) is maintained on github 

You can install Kubernetes in multiple ways, but just using the OS Distributions repo is the preferred method for me, and the instructor.

### **Building the cluster**
* install and configure packages
* create the cluster
* Configure Pod Networking
* Join Nodes to cluster

#### **Software packages**
* containerd
* kubelet
* kubeadm
* kubectl


#### **Bootstraping**

```
kubeadm init
```
* This performs pre-flight checks
* creates a CA
* Generate Kubeconfig files
* Generating static pod manifests
* wait for control plane pods
* tains the control plane node - keeps pods from running on the control plane node
* generates a bootdstrap token
* starts and add-on components/pods.

##### **kubeadm created kubeconfig files**

These files are used to define how to connect to the cluster and include 
* client certs
* cluster api server network location

/etc/kubernetes
* admin.conf (the admin account/certificate)
* all of the below are used to authenticate to the cluster for their specific needs
  * kubelet.cof 
  * controller-manager.conf 
  * scheduler.conf 

###### **CA**
The cluster procuces a self-signed CA
it CAN be a part of an external PKI, but typically its just a part of the cluster. This secures cluster communications, authentication of users and cluster components. The files live in /etc/kubernetes/pki

###### **Staic pod manifests**
Manifests describe the configuraiton of things (typically pods)
/etc/kubernetes/manifests
* etcd
* api server
* controller manager
* scheduler
the  kubelet watches this directory and any changes invoke actions.

#### **Createing a control plane node**
Download a yaml manifest of our network.

```
wget https://docs.projectcalico.org/manifests/calico.yaml
```

[calico](https://docs.projectcalico.org/manifests/calico.yaml)
The above configuration is modified to update the CIDR range for our cluster. this can be found [here](bootstrap/calico.yaml) under "CALICO_IPV4POOL_CIDR"


Use kubeadm to generate a valid cluster config, modify the [cluster config](bootstrap/ClusterConfiguration.yaml) to match our needs(optional), then init the cluster.

```
kubeadm config print init-defaults | tee ClusterConfiguration.yaml
```

```
sudo  kubeadm init --config=ClusterConfiguration.yaml --cri-socket /run/containerd/containerd.sock
```
This will output things like the IP Address:port, Token and cert hash. This information is required to join worker nodes to the cluster.

<details><summary>kubeadmin init results</summary>

```
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

kubeadm join 192.168.1.230:6443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:eafc5fe6462d3e29e0c13ce0df5f1d38bb8d31dcdf10b0803275289181b6f179
```

</details>


Next copy the credential file to the local users configuration to allow access.
```
mkdir -p $HOME/.kube
sudo cp - i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Lastly apply the network config with kubectl
```
kubectl apply -f calico.yaml
```



#### **Adding a node to a cluster**
* install packages
* kubeadm join -- network locaiton --bootstrap token -- certhash
* download cluster information
* node submits a CSR
* CA signs the CSR automatically
* Configures kubelet.conf

```
kubeadm join (ip address : port) --token (token) --discovery-token-ca-cert-hash (hash)
```


apply calico config

```
kubectl apply -f calico.yaml
```

#### **Add node to cluster**
* make sure it passes the ansible playbook
* obtain token
```
kubeadm tokenlist

TOKEN                     TTL         EXPIRES                USAGES                   DESCRIPTION                                                EXTRA GROUPS
abcdef.0123456789abcdef   23h         2022-09-12T19:13:27Z   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token

```
This token only lasts 24 hrs.
a new one can be generated with  

```
kubead token  create
```

Next we need the cert hash

```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

eafc5fe6462d3e29e0c13ce0df5f1d38bb8d31dcdf10b0803275289181b6f179
```

.... good lord.. or we can  just use this damn command do do all of that for us

```
kubeadm token create --print-join-command
```
execute teh command prefixed with "sudo" and profit....


## networking
overlay networks (software defined networking)
* flannel -layer 3 virtual network
* alico - L3 and policy based traffic management
* weave net - multi-host network

we will use calico in  this example




### ports

control plane
| Component | Ports | Used By |
| --- | --- | --- | 
| api | 6443 | All
| etcd | 2379-2380 | API/etcd |
| Scheduler | 10251 | Self |
| Controler manager | 10252 | Self |
| Kubelet | 10250 | Control Plane |

Worker nodes
| Component | Ports | Used By |
| --- | --- | --- | 
| Kubelet | 10250 | Control Plane |
| NodePort | 30000-32767 | All | 


#

## scalability

## high availability

## Disaster Recovery

## Working with the Kubernetes cluster

### using kubectl to interact with the cluster

#### opperations
kubectl - primary tool
* operations
  * apply/create - creates a resource
  * run - start a pod from an image (ad hoc)
  * explain - documentation for the resource
  * delete - deletes a resource
  * get - list resources
  * describe - deteiled resource information
  * exec - execute a command on a container (like docker exec)
  * logs - view logs on a container - valueable troubleshooting

#### resources

* resources
  * nodes (no)
  * pods (po)
  * services (svc)

#### output
* output
  * format
    * wide - output additional info
    * yaml - YAML formatted API object
    * json - JSON formatted API object
    * dry-run - print an objecct without sending it to the API server (skelliton)
* 

#### kubectl

```
kubectl [command] [type] [name] [flags]

kubectl get pods pod1 --output=yaml
kubectl create deployment nginx --image=nginx
```

configure autocomplete
```
echo "source <(kubectl completion bash)" >> ~/.bashrc
source ~/.bashrc
```

#### demo



### application deployments INTO the cluster

####imperative configuration

this is one object at a time...
```
kubectl create deployment nginx --image=nginx
```

```
kubectl run nginx --image=nginx
```

CLI isnt sustainable

#### declaritive
define our desired state in code
manifest

##### our manifest (manual review)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
        - image: gcr.io/google-samples/hello-app:1.0
          name: hello-app

```

kubectl apply -f deployment.yaml

##### generating a manifest with a dry run (automatic and great)
```
kubectl create deployment hello-world --image=gcr.io/google-samples/hello-app:1.0 --dry-run=client -o yaml > deployment.yaml
```

NOTE!!! Found that the super basic hello-world samples do NOT support ARM processors :0p. switched to good ole nginx.

To list out the running containers on any cluster member.

```
sudo crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps
```

attaching a shell to a pod

```
kubectl exec -it hello-world-pod -- /bin/bash
```


##### expose application

```
kubectl expose deployment nginx-6d666844f6-rrjjn --port=80 --target-port=80
```

```
ansible@autobot:~/git/SmitKub/appdeploy$ kubectl describe service nginx
Name:              nginx
Namespace:         default
Labels:            app=nginx
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.99.122.48
IPs:               10.99.122.48
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         192.168.4.2:80
Session Affinity:  None
Events:            <none>
```

```
ansible@autobot:~/git/SmitKub/appdeploy$ curl http://10.99.122.48:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```


##### declaritive deployment
applicaiton
```
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml | more
```

service

```
kubectl expose deployment nginx --port=80 --target-port=80 --dry-run=client -o yaml > service.yaml
```

```
kubectl apply -f service.yaml
```

to update an  application,  update the manifest and then re-apply with kubectl

```
kubectl apply -f deployment.yaml
```


# 9-26-2022
pickingup
navigated to /git/SmitKube/appdeploy/hello-world

had to 
```
kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx --port=80 --target-port=80 --dry-run=client -o yaml > service.yaml

kubectl apply -f service.yaml
```

how do we expose the app on the routeable network.
```
kubectl expose deployment nginx --port=80 --target-port=80 --type=NodePort --dry-run=client -o yaml > service.yaml
```
# successfull exposure
ooookaaay...
per this [helpful article](https://medium.com/swlh/kubernetes-external-ip-service-type-5e5e9ad62fcd) The app is successfully exposed by leveraging the control node's ip address in the configuration. I suspect that if i configure a service such as keepalived on multiple control nodes, the issue of "HA" would be resolved.
in the meantime i am going to try and provision some specific IP addrsses on the control node and see if i can get nginx to follow it.

```
ansible@autobot:~/git/SmitKub/appdeploy/hello-world$ cat service.yaml 
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  externalIPs:
    - 192.168.1.230
status:
  loadBalancer: {}
```

```
Current DNS Server: 192.168.1.231
       DNS Servers: 192.168.1.231 192.168.1.1
        DNS Domain: smithsonite.home
```

## confirmed
Confirmed working when a specific ip address (192.168.1.235) was assigned to the control node

### netplan config

```
ansible@autobot:~/git/SmitKub/appdeploy/hello-world$ cat /etc/netplan/50-cloud-init.yaml
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    version: 2
    wifis:
        renderer: networkd
        wlan0:
            access-points:
                CampSmit:
                    hidden: true
                    password: this is a secret...
            dhcp4: no
            addresses: [192.168.1.230/24, 192.168.1.235/24]
            routes:
                - to: default
                  via: 192.168.1.1
            nameservers:
              addresses: [192.168.1.231, 192.168.1.1]
              search: [smithsonite.home]
            optional: true
```
then use

```
sudo netplan apply
```

## cleanup
```
kubectl delete service nginx
kubectl delete deployment nginx
```



# calico and external IP
https://projectcalico.docs.tigera.io/networking/advertise-service-ips

ok.. so nodeport will require a unique ip/port combination for all services...

INGRESS will allow us to use a reverse proxy.. such as NGINX.
Ingress looks like what i want to setup

it LOOKS like ingress services just get a public ip.

so ingress deployment > service > application deployment.


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



# **Resources**
https://app.pluralsight.com/library/courses/kubernetes-installation-configuration-fundamentals/table-of-contents

https://app.pluralsight.com/library/courses/configuring-managing-kubernetes-networking-services-ingress/table-of-contents