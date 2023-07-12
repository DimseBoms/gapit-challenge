# My notes for gapit's application challenge

## 1. Virtualbox setup
Since I'm running a Linux distribution I chose to install virtualbox via my package manager, which is Zypper as I am on openSUSE Tumbleweed.
```
sudo zypper in virtualbox
```
I also had to add my user to the vboxusers group to be able to use virtualbox
```
sudo usermod -aG vboxusers dmitriy
```

## 2. Virtual machine setup
I downloaded the ISO and performed a manual install. Since this is my private virtual machine I did not add any domain.

## 3. Setting up static IP and communication between virtual machines
To enable communication between gapit-master and gapit-client I configured the boxes to use the bridged adapter from the virtualbox network settings ([trainenv.blogspot.com](http://trainenv.blogspot.com/2016/03/virtual-networking.html)), ([virtualbox.org](https://www.virtualbox.org/manual/ch06.html#network_bridged)). This ensures that the boxes can see each other as virtualbox will simulate a physical network connection with my host machine, making them both visible to my host machine's local network.

To assign static IP addresses to the boxes I made a netplan YAML config file as described in the [Ubuntu server documentation](https://ubuntu.com/server/docs/network-configuration). To get the gateway and routing table I had to run the `ip route` command while the box was running in DHCP mode:
```
gapit-client@gapit-client:~$ ip route
default via 192.168.205.98 dev enpOs3 proto static
default via 192.168.205.98 dev enpOs3 proto dhcp src 192.168.205.150 metric 100 192.168.205.0/24 dev enpOs3 proto kernel scope link src 192.168.205.151
192.168.205.986 dev enpos3 proto dhcp scope link src 192.168.205.150 metric 100
```

Then I created a config file that netplan could use `/etc/netplan/99_config.yaml`:
gapit-master:
```
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.205.150/24 # static IP to be assigned
      routes:
        - to: default
          via: 192.168.205.98 # gateway
      nameservers:
          addresses: [8.8.8.8, 1.1.1.1]
```
gapit-client:
```
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.205.151/24 # static IP to be assigned
      routes:
        - to: default
          via: 192.168.205.98 # gateway
      nameservers:
          addresses: [8.8.8.8, 1.1.1.1]
```

And applied it with:
```
sudo netplan apply
```
