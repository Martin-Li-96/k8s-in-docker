路由器: 192.168.2.0
物理机和容器处于同一网络

# Configure ipvlan in docker
```bash
docker network create -d ipvlan \
--subnet=192.168.2.0/24 \
--gateway=192.168.2.1 \
-o ipvlan_model=l2 \
-o parent=enp0s5 k8s-vlan1
```
# Run docker with static ip in ipvlan
```bash
docker run -idt \
--network=k8s-vlan1 \
--ip 192.168.2.200 \
--name k8smaster1 ubuntu /bin/bash
```



# Script of build k8smaster1...
copy the following script to a .sh file
```bash
#!/bin/bash

# Master 1
master1_ip=192.168.2.200
docker run -idt --network=k8s-vlan1 --ip $master1_ip --name k8smaster1 ubuntu /bin/bash           
docker cp /etc/passwd k8smaster1:/etc/passwd 
docker cp /etc/shadow k8smaster1:/etc/shadow
docker cp /etc/group k8smaster1:/etc/group      
docker exec -d k8smaster1 /bin/bash -c "apt update;apt upgrade -y;apt install net-tools -y;apt install sudo -y; apt install openssh-server -y;service ssh start;apt install iputils-ping -y;"
#Master 2
master2_ip=192.168.2.201
docker run -idt --network=k8s-vlan1 --ip $master2_ip --name k8smaster2 ubuntu /bin/bash           
docker cp /etc/passwd k8smaster2:/etc/passwd 
docker cp /etc/shadow k8smaster2:/etc/shadow
docker cp /etc/group k8smaster2:/etc/group      
docker exec -d k8smaster2 /bin/bash -c "apt update;apt upgrade -y;apt install net-tools -y;apt install sudo -y; apt install openssh-server -y;service ssh start;apt install iputils-ping -y;"
#Master 3
master3_ip=192.168.2.202
docker run -idt --network=k8s-vlan1 --ip $master3_ip --name k8smaster3 ubuntu /bin/bash           
docker cp /etc/passwd k8smaster3:/etc/passwd 
docker cp /etc/shadow k8smaster3:/etc/shadow
docker cp /etc/group k8smaster3:/etc/group      
docker exec -d k8smaster3 /bin/bash -c "apt update;apt upgrade -y;apt install net-tools -y;apt install sudo -y; apt install openssh-server -y;service ssh start;apt install iputils-ping -y;"
```

You can use following command to check whether openssh-server is installed successfully. Once you can see "ssh[+]", which means the ssh server is installed.
```bash
docker exec -t k8smaster1 bash service --status-all
```