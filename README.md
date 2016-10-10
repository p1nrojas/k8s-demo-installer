#Prepare your #Kubernetes Cluster with #Ansible and #cloud_init

A bit tired using bash scripts. I’ve done this small playbook based on the post: <A HREF="https://pinrojas.com/2016/08/01/resize-and-manage-cloud-init-on-kvm-with-centos-cloud-images/">“Resize and manage cloud-init on kvm with centos cloud images”</A>. to build up your k8s master/minions over KVM using a Cloud QCOW2 image. I’ve used cloud_init to configure those cloud images from a temporary cdrom iso.

Check out the code at my repo: p1nrojas/k8s-demo-installer

##Requirements

Check out you have enough memory and cpu to share among the instances.
Check out your NS record are working out for the server you will create
Download the latest centos cloud image version:

```
curl http://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2 > /tmp/centos.qcow2
```
Install libvirt, libguestfs-tools and genisoimage. I’ve added a task to check those packages out anyway:

```
- name: Install additional packages to modify qcow image and run as VM - RedHat Host
  yum: name={{ item }} state=present
  with_items:
   - qemu-kvm
   - libvirt
   - bridge-utils
   - libguestfs-tools
   - libvirt-python
   - genisoimage
  when: ansible_os_family == "RedHat"
  delegate_to: "{{ hypervisor }}"
```

##Configuration

Fix the inventory as you wish. Set as many master and minions you want (and your server is capable to support). Remember this is working into just one hypervisor.
However, you can change that in the host_vars to address as many you need.

```
# inventory file
[masters]
master.nuage.lab
 
[minions]
minion01.nuage.lab
minion02.nuage.lab
```

Start doing changes at group_vars at all.yml. Also you can split up the setup per group of minions or masters (i.e. minions.yml). Parameters to setup are:

- Bridges: every server would have two interfaces to the same/different bridges. This case I am using the same for data and management interfaces
- Network settings: Network mask to every interface type, DNS, domain, default gateway, ntp-servers (I am actually saving this latter for the future)
- SSH public keys would be used later to setup the K8s later.Check my post <A HREF="https://pinrojas.com/2016/08/30/local-install-of-kubernetes-on-centos7-kvm/">#Kubernetes in a few steps</A> and <A HREF="https://pinrojas.com/2016/08/31/most-scalable-kubernetes-install-with-nuage-installation-notes/">Scale out #Kubernetes with #Nuage: 
Installation notes</A>
```
# all.yml in group_vars
if_mgmt:
  linux_bridge: bridge0
  mask: 255.255.255.0
if_data:
  linux_bridge: bridge0
  mask: 255.255.255.0
 
dns:
  servers:
    - 10.10.10.2
  domain: nuage.lab
 
ntp_servers:
  - 10.10.10.2
 
gateway: 10.10.10.1
 
# Add your ssh keys to access all servers later
ssh_rsa:
  key:
  - "ssh-rsa XOXOXOXOXOXOXOXOOXOXOXOOXOXOXOXOXOXOXOXOOXOXOXOXOXOXOXOXOXOXOXOXXXOjQng3P3e/C5ezpPqnvLZO2jq2ivdVXvjKNisIop3CzymRD7tf4wglDiREggSt2fPXXhIMG7AIbQuvn88XZ+PCa3Y74L8G9u8WWJMZzY5fU4I7UP3SZb2FVxLXC7iTukYM8QRMhbwkAVzZWOj81jGRZI/9b/zU+jm2IFPPyTNdMbyXlfQ9qMbWA11PxdP4G5AcMcHmGKT9 root@box01.nuage.lab"
  - "ssh-rsa AAAAB3NzaC1ycB8K3VUXOXOXOXOXOXOXOXOXOXOXOXOXOXOXOOXOXOXOXOXOXvfrrGg9fg8som/S9bt1sdRUUHuHkgfm9ZRIlG8ATS15ZOmfHrvT4KfRcY9NbiR7loITxLs5W7lCH5YtZ8AFp0HjzjwiaEnhVB3+O2iYeFs+2cwbNvENnKRzl7ZgEeCRYKbS+OcAOmk0+rGBx7rHTSg+MfkLtX3VgfNdUxx+ZKeAMqDkSuKSTlOZJDjIbAW0pCffp mauricio.rojas@nuagenetworks.net"
Next, let’s go the host_vars
```
Define the memory, cpu and disk values (this could be added into the group_vars i.e. for minions.yml if all would have the same capacities). Memory is in KiB. Set the IP address for the interfaces.
```
# host_vars file: minions01.nuage.lab.yml
hypervisor: 192.168.1.113
memory_KiB: 16777216
vcpu_num: 8
disk_GB: 50
interfaces:
  mgmt:
    ip: 10.10.10.102
  data:
    ip: 10.20.10.102
```    
##Templates

There are some files you would want to take a look in k8s-demo-installer/roles/cluster/templates/

- nodes.xml.j2 – XML file used to define the KVM domain for every server. You may want to add some additional settings to your VM thru it
- meta-data.j2 – cloud_init file to set up your hosts and connection settings
- user-data.j2 – cloud_init file to set up your network settings
See ya!

check more details at pinrojas.com
