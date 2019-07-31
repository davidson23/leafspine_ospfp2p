Prerequisites
1.Make sure Vagrant is installed in the server.
2.Make sure Virtualbox is installed in the server.
3.Make sure that there is adequate space in your machine.There are 6 VMs and each VM is configured with 1G RAM capacity.You can also try to bring-up each VM with 512MB RAM by doing the appropirate changes in the Vagrantfile.



Procedure
1.Untar the file in the server where you want to install the setup.Here i have used a Ubuntu 16.04 desktop image for demostration.

2.You will find the vagrant file with it's default name "Vagrantfile".Bring up the setup using the below command,

vagrant up

3.Once all the VMs are activated,you can login to each VM using the below command,

vagrant ssh L1

Similarly do for other VMs(L2,S1,S2,PC1 and PC2)

4.There is a file in this directory(configuration.txt) which has the configurations for the individual switches.Login to each switch and execute the commands.

5.Once all the configuration are done,you will be able to ping PC2 from PC1 and vice-versa.


root@pc1:~# ping 192.168.2.1
PING 192.168.2.1 (192.168.2.1) 56(84) bytes of data.
64 bytes from 192.168.2.1: icmp_seq=1 ttl=61 time=5.72 ms
64 bytes from 192.168.2.1: icmp_seq=2 ttl=61 time=3.49 ms
64 bytes from 192.168.2.1: icmp_seq=3 ttl=61 time=2.72 ms
64 bytes from 192.168.2.1: icmp_seq=4 ttl=61 time=2.87 ms


root@pc2:~# ping 192.168.1.1
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=61 time=2.99 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=61 time=3.19 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=61 time=3.16 ms
