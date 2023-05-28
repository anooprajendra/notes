## Setup Virtual Machines

1. Setup Bare Virtual Machine with the following specs
    - 4 GB of RAM
    - 20 GB HDD
    - 4 CPU - Host Passthrough Mode
    - Single Network Interface connected to the **default** NAT network
1. Install Debian 11.7.0 on it using Debian's Network installer.
   > When installing the OS, create a user called `k0s`
1. Turn off machine once it completes installation
1. Clone VM 3 times so create 3 more Virtual Machines
1. Boot servers.

## Setup Ansible controller node.
1. Pick a host to be the ansible controller node. You can choose to use the
   hypervisor or another VM controller node.
1. Add IP addresses of all hosts to `/etc/hosts` file on the controller node
   1. First, get the IP addresses from the hypervisor using the `virsh` command
      ```sh
      # virsh net-dhcp-leases default
      Expiry Time           MAC address         Protocol   IP address           Hostname    Client ID or DUID
      -----------------------------------------------------------------------------------------------------------------------------------------------
      2023-05-27 17:39:21   52:54:00:ca:88:1a   ipv4       192.168.122.61/24    k0s-node0   ff:00:ca:88:1a:00:01:00:01:2c:05:54:61:52:54:00:ca:88:1a
      2023-05-27 18:03:01   52:54:00:a2:2d:c7   ipv4       192.168.122.58/24    k0s-node1   ff:00:a2:2d:c7:00:01:00:01:2c:05:54:61:52:54:00:ca:88:1a
      2023-05-27 18:07:07   52:54:00:b2:66:0c   ipv4       192.168.122.109/24   k0s-node2   ff:00:b2:66:0c:00:01:00:01:2c:05:54:61:52:54:00:ca:88:1a
      2023-05-27 18:09:09   52:54:00:3d:33:8e   ipv4       192.168.122.138/24   k0s-node3   ff:00:3d:33:8e:00:01:00:01:2c:05:54:61:52:54:00:ca:88:1a
      ```
   1. The output shows the IP addresses, hostnames, and MAC addresses of the hosts created. Use this information
      to append the following to the `/etc/hosts` file on the controller node.
      ```
      192.168.122.61  k0s-node0.domain k0s-node0
      192.168.122.58  k0s-node1.domain k0s-node1
      192.168.122.109 k0s-node2.domain k0s-node2
      192.168.122.138 k0s-node3.domain k0s-node3
      ```
1. Create an SSH Key using `ssh-keygen`. Use default values
1. Copy the SSH-key to all the nodes, so as to enable passwordless SSH access
   ```bash
   ssh-copy-id k0s@k0s-node0
   ssh-copy-id k0s@k0s-node1
   ssh-copy-id k0s@k0s-node2
   ssh-copy-id k0s@k0s-node3
   ```
1. On the hypervisor, or another controller virtual machine, install **Ansible**.
1. Create an Ansible **inventory** file with the following contents
   ```yaml
   all:
     hosts:
       k0s-node0:
         ansible_user:k0s
       k0s-node1:
         ansible_user:k0s
       k0s-node2:
         ansible_user:k0s
       k0s-node3:
         ansible_user:k0s
   ```
1. Test Ansible to ensure connectivity.
   ```bash
   # ansible '*' -m command -a hostname
   k0s-node3.heli0.xyz | CHANGED | rc=0 >>
   k0s-node3
   k0s-node0.heli0.xyz | CHANGED | rc=0 >>
   k0s-node0
   k0s-node1.heli0.xyz | CHANGED | rc=0 >>
   k0s-node1
   k0s-node2.heli0.xyz | CHANGED | rc=0 >>
   k0s-node2
   ```
