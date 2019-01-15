# KVM-CheatSheet

## Setup DNS KVM VMs

Edit hosts file to add DNS entries on the KVM Server:
```
sudo nano /etc/hosts
```
Add lines for DNS resolution entries:
> 192.168.123.211    kube01.home.lab
    192.168.123.212    kube02.home.lab

## Setup DHCP Leases on KVM server for VMs
```
# Edit DHCP leases
sudo virsh net-edit default
```
add DHCP lease under <range start ..... />
> <dhcp>
      <range start='192.168.111.100' end='192.168.111.254' />
      <host mac='52:54:00:26:61:22' name='virt.example.com' ip='192.168.111.111' />
    </dhcp>
```
# Save settings:
sudo virsh net-destroy default && virsh net-start default

# To flush DNS:
sudo systemd-resolve --flush-caches
```
... **Changes may require power off and on of VMs after changes** ...