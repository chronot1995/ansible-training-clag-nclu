## int-ansible-training-clag-nclu

### Summary:

This is an Ansible demo which configures two Cumulus VX switches in a CLAG configuration with a Linux server. This demo will utilize the Ansible Cumulus NCLU module.

### Network Diagram:

![Network Diagram](https://github.com/chronot1995/int-ansible-training-clag-nclu/blob/master/documentation/int-ansible-training-clag-nclu.png)

### Initializing the demo environment:

First, make sure that the following is currently running on your machine:

1. Vagrant > version 2.1.2

    https://www.vagrantup.com/

2. Virtualbox > version 5.2.16

    https://www.virtualbox.org

3. Copy the Git repo to your local machine:

    ```git clone https://github.com/chronot1995/int-ansible-training-clag-nclu```

4. Change directories to the following

    ```int-ansible-training-clag-nclu```

6. Run the following:

    ```./start-vagrant-poc.sh```

### Running the Ansible Playbook

1. SSH into the oob-mgmt-server:

    ```cd vx-simulation```   
    ```vagrant ssh oob-mgmt-server```

2. Copy the Git repo unto the oob-mgmt-server:

    ```git clone https://github.com/chronot1995/int-ansible-training-clag-nclu```

3. Change directories to the following

    ```int-ansible-training-clag-nclu/automation```

4. Run the following:

    ```./provision.sh```

This will bring run the automation script and configure the two switches with CLAG.

### Troubleshooting

Helpful NCLU troubleshooting commands:

- net show clag
- net show interface bonds
- net show interface bondmems
- net show route
- net show interface | grep -i UP
- net show lldp

Helpful Linux troubleshooting commands:

- ip route
- ip link show
- ip address <interface>
- cat /proc/net/bonding/uplink

The CLAG status command will verify the CLAG peer status:

```
cumulus@switch01:mgmt-vrf:~$ net show clag status
The peer is alive
     Our Priority, ID, and Role: 100 44:38:39:00:00:05 primary
    Peer Priority, ID, and Role: 100 44:38:39:00:00:06 secondary
          Peer Interface and IP: peerlink.4094 169.254.1.2
                      Backup IP: 192.168.200.2 vrf mgmt (active)
                     System MAC: 44:38:39:ff:01:56

CLAG Interfaces
Our Interface      Peer Interface     CLAG Id   Conflicts              Proto-Down Reason
----------------   ----------------   -------   --------------------   -----------------
          bond01   -                  1         -                      -
```

One can see the various LACP interfaces:

```
cumulus@switch01:mgmt-vrf:~$ net show interface bondmems
    Name    Speed      MTU  Mode     Summary
--  ------  -------  -----  -------  --------------------
UP  swp1    1G        1500  LACP-UP  Master: bond01(DN)
UP  swp2    1G        1500  LACP-UP  Master: peerlink(UP)
UP  swp3    1G        1500  LACP-UP  Master: peerlink(UP)
```

One can also view the MAC addresses of the two switches within the EVPN instance by running the following command:

```
cumulus@switch01:mgmt-vrf:~$ net show interface bonds
    Name      Speed      MTU  Mode    Summary
--  --------  -------  -----  ------  --------------------------------
DN  bond01    N/A       1500  LACP    Bond Members: swp1(UP)
UP  peerlink  2G        1500  LACP    Bond Members: swp2(UP), swp3(UP)
```

```
cumulus@switch01:mgmt-vrf:~$ ip address show | grep vlan100
40: vlan100@bridge: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    inet 172.16.121.2/24 scope global vlan100
41: vlan100-v0@vlan100: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    inet 172.16.121.1/24 scope global vlan100-v0
```

```
cumulus@server01:~$ cat /proc/net/bonding/uplink | grep Status
MII Status: up
MII Status: up
MII Status: up
```


### Errata

1. To shutdown the demo, run the following command from the vx-simulation directory:

    ```vagrant destroy -f```

2. This topology was configured using the Cumulus Topology Converter found at the following URL:

    https://github.com/CumulusNetworks/topology_converter

3. The following command was used to run the Topology Converter within the vx-simulation directory:

    ```python2 topology_converter.py int-ansible-training-clag-nclu.dot -c```

    After the above command is executed, the following configuration changes are necessary:

4. Within ```vx-simulation/helper_scripts/auto_mgmt_network/OOB_Server_Config_auto_mgmt.sh```

The following stanza:

    #Install Automation Tools
    puppet=0
    ansible=1
    ansible_version=2.3.1.0

Will be replaced with the following:

    #Install Automation Tools
    puppet=0
    ansible=1
    ansible_version=2.6.2

The following stanza will replace the install_ansible function:

```
install_ansible(){
echo " ### Installing Ansible... ###"
apt-get install -qy ansible sshpass libssh-dev python-dev libssl-dev libffi-dev
sudo pip install pip --upgrade
sudo pip install setuptools --upgrade
sudo pip install ansible==$ansible_version --upgrade
}```

Add the following ```echo``` right before the end of the file.

    echo " ### Adding .bash_profile to auto login as cumulus user"
    echo "sudo su - cumulus" >> /home/vagrant/.bash_profile
    echo "exit" >> /home/vagrant/.bash_profile

    echo "############################################"
    echo "      DONE!"
    echo "############################################"
