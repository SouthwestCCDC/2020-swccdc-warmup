# OVA credentials
- username: `root`
- password: `ccdc`

# oneadmin credentials
- username: `oneadmin`
- password: `93762290fadc4665338878b8fee76d5c`

# minimum system requirements
- 32GB available disk
- 7GB available RAM

# opennebula vm ubuntu 20.04 template instance credentials
- username: `root`
- password: `ccdc`

# provisioning notes (for black team)
- configure networking for the `opennebula-frontend` vm
  - select settings -> network -> adapter 1
    - set to NAT
  - select adapter 2
    - set to internal network with name: `swccdc-warmup2020-internal`
- execute `VBoxManage modifyvm "opennebula-frontend" --natpf1 "sunstone,tcp,127.0.0.1,9869,,9869"`
- execute `VBoxManage modifyvm "opennebula-frontend" --natpf1 "sunstone-ssh,tcp,127.0.0.1,8022,,22"`
- execute `VBoxManage modifyvm "opennebula-frontend" --natpf1 "sunstone-vnc,tcp,127.0.0.1,29876,,29876"`
- execute `VBoxManage modifyvm "opennebula-hypervisor" --natpf1 "hypervisor-ssh,tcp,127.0.0.1,9022,,22"`
- boot `opennebula-frontend` and login
  - execute `hostnamectl set-hostname opennebula-frontend`
  - execute `echo "10.235.59.1 opennebula-frontend" >> /etc/hosts`
  - execute `echo "10.235.59.2 opennebula-hypervisor" >> /etc/hosts`
- configure networking for the `opennebula-hypervisor` vm
  - select settings -> network -> adapter 1
    - set to NAT
  - select adapter 2
    - set to internal network with name: `swccdc-warmup2020-internal`
- execute `VBoxManage modifyvm "opennebula-hypervisor" --nested-hw-virt on`
- boot `opennebula-hypervisor` and login
  - use nmtui to create a bridge interface named br0 with enp0s3 set as a slave
    - set to link-local for ipv4 and ipv6
  - execute `hostnamectl set-hostname opennebula-hypervisor`
  - execute `echo "10.235.59.1 opennebula-frontend" >> /etc/hosts`
  - execute `echo "10.235.59.2 opennebula-hypervisor" >> /etc/hosts`
- from your provisioning shell, run ansible
  - `ansible-playbook -i hosts -u root --ask-pass deploy-opennebula.yml`
- login to the frontend and hypervisor hosts and execute the following on each
  - `su oneadmin`
  - `ssh opennebula-frontend`
  - `ssh opennebula-hypervisor`
  - `exit`
- open a browser and browse to http://127.0.0.1:8996
  - create a network named `external-network`
    - bridge: `br0`
    - network mode: `bridged`
    - physical interface: `enp0s3`
    - first address: `10.0.2.200`
    - size: `10`
    - network address: `10.0.2.0`
    - network mask: `255.255.255.0`
    - gateway: `10.0.2.2`
    - dns: `10.0.0.2`
    - mtu of guests: `1500`
  - create a network named `internal-network`
    - network mode: `vxlan`
    - physical device: `enp0s8`
    - first address: `172.16.4.1`
    - size: `100`
    - network address: `172.16.4.0`
    - network mask: `255.255.255.0`
    - mtu of guests: `1000`

# usage notes (for blue team)
- download and install VirtualBox
- download the ovas
- import all ovas into virtual box
- select the opennebula-frontend
  - select settings -> network -> adapter 1
    - set to NAT
  - select adapter 2
    - set to internal network with name: `swccdc-warmup2020-internal`
- execute `VBoxManage modifyvm "opennebula-frontend" --natpf1 "sunstone,tcp,127.0.0.1,9869,,9869"`
- execute `VBoxManage modifyvm "opennebula-frontend" --natpf1 "sunstone-ssh,tcp,127.0.0.1,8022,,22"`
- execute `VBoxManage modifyvm "opennebula-hypervisor" --natpf1 "hypervisor-ssh,tcp,127.0.0.1,9022,,22"`
- boot `opennebula-frontend`
- boot `opennebula-hypervisor`
- open a browser and browse to http://127.0.0.1:8996
  - create a virtual router by creating a new vm from the `Service VNF` template
    - enable dns server
      - listen on eth1
    - enable nat
      - outgoing interface on eth0
    - enable router
      - router interface eth0,eth1
    - eth0 attached to `external-network`
    - eth1 attached to `internal-network`
  - create an ubuntu 20.04 instance
    - eth0 attached to `internal-network`
  - open a vnc for the ubuntu instance. It should have an address of `172.16.4.2`
    - check connectivity by pinging google.com