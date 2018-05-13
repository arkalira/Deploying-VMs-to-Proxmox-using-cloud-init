# Deploying VMs to Proxmox using cloud-init

## Cloud init provisioning in Proxmox PVE

- VM: Debian 9
- Proxmox: 5.1

## Download cloud init cdimage for Debian 9

- Download to local folder in Proxmox

```
wget https://cdimage.debian.org/cdimage/openstack/current-9/debian-9-openstack-amd64.qcow2
```

## Configure VM Settings using qm commands

- Now we are going toc reate a template vm with the following settings:

 - Debian 9
 - 1G RAM
 - 1 CPU
 - 1 Network interface with static ip address

```
qm create 9000 --name debian-cloud-image --memory 1024 --net0 virtio,bridge=vmbr0,tag=10 --cores 1 --sockets 1 --cpu cputype=kvm64 --description "Debian 9.4 cloud image" --kvm 1 --numa 1
qm importdisk 9000 debian-9-openstack-amd64.qcow2  datos
qm set 9000 --scsihw virtio-scsi-pci --virtio0 datos:vm-9000-disk-1
qm set 9000 --serial0 socket
qm set 9000 --boot c --bootdisk virtio0
qm set 9000 --agent 1
qm set 9000 --hotplug disk,network,usb,memory,cpu
qm set 9000 --vcpus 1
qm set 9000 --vga qxl
qm set 9000 --name glusterfs-micro
#Cloud INIT
qm set 9000 --ide2 datos:cloudinit
qm set 9000 --sshkey /etc/pve/pub_keys/pub_key.pub
qm set 9000 --ipconfig0 ip=10.200.1.220/24,gw=10.200.1.1
```

- IMPORTANT: Before start you new VM, resize the disk using Proxmox menu to fit your needings.
- Once you have done that, you can start your VM 9000 and install all of the needed software (git, docker, glusterfs, etc).
- When the template vm is ready, shutdown and clone it as many times as needed using the following script.

## Crear VMs

- Creation of 3 VMs to use as gluster servers from the previous template VM 9000.

```
for ID in 1 2 3
do
VMID="80$ID"
qm clone 9000 $VMID --name glusterdsp0$ID
qm set $VMID --name glusterfs-micro0$ID
qm set $VMID --net0 model=virtio,bridge=vmbr0,tag=10
qm set $VMID --ipconfig0 ip=10.200.1.12$ID/24,gw=10.200.1.1
qm set $VMID --searchdomain yourdomain.com
qm set $VMID --nameserver 10.200.1.1
qm set $VMID --onboot 1
qm start $VMID
done
```

- And its done! Enjoy
