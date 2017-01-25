#Prepare your #Kubernetes Cluster with #Ansible and #cloud_init

A bit tired using bash scripts. I’ve done this small playbook based on the post: <A HREF="https://pinrojas.com/2016/08/01/resize-and-manage-cloud-init-on-kvm-with-centos-cloud-images/">“Resize and manage cloud-init on kvm with centos cloud images”</A>. to build up your k8s master/minions over KVM using a Cloud QCOW2 image. I’ve used cloud_init to configure those cloud images from a temporary cdrom iso.

Check out the code at my repo: p1nrojas/k8s-demo-installer

##Requirements

Check out you have enough memory and cpu to share among the instances.
Check out your NS record are working out for the server you will create
Download the latest centos cloud image version:

```
curl http://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2 > /tmp/centos7.qcow2
```
Install libvirt, libguestfs-tools and genisoimage. I’ve added a task to check those packages out anyway

Create one or two bridges (management and data networks) at your hypervisors

##Quick Start

Just execute:

```
ansible-playbook build.yml
ansible-playbook -i hosts cluster-install.yml
```

##Configuration

Check and use build.yml to create your inventory and vars files (host and group). 
Inside build.yml you will define IPs, netmasks, gateways, ssh keys, VM's resources....

After play build.ym you will get a hosts inventory file like this:
```
# inventory hosts file
[masters]
master.nuage.lab
 
[minions]
minion01.nuage.lab
minion02.nuage.lab
```

Then you can play cluster-install.yml

You also have the option to reset everything using cloud-reset.yml and then build-reset.yml (Last one will create a build.bak file with your last settings)
See ya!

