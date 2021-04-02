# kubernetes-and-ansible
This repository has set of ansible playbooks created to setup a kubernetes cluster fully automated with one master and multiple worker nodes. This will work on physical servers, virtual machines, aws cloud, google cloud or any other cloud servers. This has been tested and verified on Centos 7.3 64 bit operating systems.

How to use this (Setup Instructions):
1. Make your servers ready (one master node and multiple worker nodes).

            Hostname                   IP             CPUs     RAM      OS          Role
            kubernetes-master    192.168.122.171        2      2 GB     CentOs-7    Master-Node
            kubernetes-node1     192.168.122.158        2      2 GB     CentOs-7    Worker-Node
            kubernetes-node2     192.168.122.225        2      2 GB     CentOs-7    Worker-Node
      
            Note : Above Lab Setup is According to my Laptop Configuration. You can Change as per Your requirements.
       

2. Make an entry of your each hosts in /etc/hosts file for name resolution.

            [root@kubernetes-master ~]# vim /etc/hosts
            192.168.122.171  kubernetes-master
            192.168.122.158  kubernetes-node1
            192.168.122.225  kubernetes-node2
             
:wq (save and exit)

3. Make sure kubernetes master node and other worker nodes are reachable between each other.
 
             [root@kubernetes-master ~]# ping kubernetes-node1
             [root@kubernetes-master ~]# ping kubernetes-node2
Ping from worker nodes also

4. Make sure git package installed on your Master and Nodes,Internet connection must be enabled in all nodes, required packages will be downloaded from kubernetes official yum repository.

            [root@kubernetes-master ~]# yum install git -y

5. Clone this repository into your master node.
 
            [root@kubernetes-master ~]# git clone https://github.com/yashpande15/kubernetes-and-ansible.git
               
   once it is cloned, get into the directory
   
            [root@kubernetes-master ~]# cd kubernetes-and-ansible

6. There is a file "hosts" available in "centos" directory, Just make your entries of your all kubernetes nodes.
 
            [root@kubernetes-master kubernetes-and-ansible]# vim centos/hosts
            [kubernetes-master-nodes]
            kubernetes-master ansible_host=192.168.122.171

            [kubernetes-worker-nodes]
            kubernetes-node1 ansible_host=192.168.122.158
            kubernetes-node2 ansible_host=192.168.122.225
            
            Note : Change above information as per your Setup.
    :wq (save and exit)        

7. Provide your server details in "env_variables" available in "centos" directory.

            [root@kubernetes-master kubernetes-and-ansible]# vim centos/env_variables
            #Edit these values only as per your environment
            #Enter your master node advertise ip address and cidr range for the pods.
            ad_addr: 192.168.122.171
            cidr_v: 172.16.0.0/16

            ###################################################################################
            # Dont Edit these below values, these are mandatory to configure kubernetes cluster
            packages:
            - docker
            - kubeadm
            - kubectl

            services:
            - docker
            - kubelet
            - firewalld

            ports:
            - "6443/tcp"
            - "10250/tcp"

            token_file: join_token
            ###################################################################################
            # Dont Edit these above values, these are mandatory to configure kubernetes cluster
    :wq (save and exit)

8. Deploy the ssh key from master node to other nodes for password less authentication.

            [root@kubernetes-master ~]# ssh-keygen
     
     Copy the public key to all nodes including your master node and make sure you are able to login into any nodes without password.
     
            [root@kubernetes-master ~]# ssh-copy-id root@kubernetes-master
            [root@kubernetes-master ~]# ssh-copy-id root@kubernetes-node1
            [root@kubernetes-master ~]# ssh-copy-id root@kubernetes-node2

9. Run "settingup_kubernetes_cluster.yml" playbook to setup all nodes and kubernetes master configuration.
 
          [root@kubernetes-master ~]# ansible-playbook settingup_kubernetes_cluster.yml

10. Run "join_kubernetes_workers_nodes.yml" playbook to join the worker nodes with kubernetes master node once "settingup_kubernetes_cluster.yml" playbook tasks are completed.
 
         [root@kubernetes-master ~]# ansible-playbook join_kubernetes_workers_nodes.yml

11. Verify the configuration from master node.

         [root@kubernetes-master ~]# kubectl get nodes
         NAME                  STATUS    ROLES     AGE     VERSION
         kubernetes-master     Ready     Master    1m30s   v1.13.1 
         kubernetes-node1      Ready     <none>    34s     v1.13.1
         kubernetes-node2      Ready     <none>    33s     v1.13.1
     
What are the files this repository has?:

ansible.cfg - Ansible configuration file created locally.

hosts - Ansible Inventory File

env_variables - Main environment variable file where we have to specify based on our environment.

settingup_kubernetes_cluster.yml - Ansible Playbook to perform prerequisites ready, setting up nodes, configure master node.

configure_worker_nodes.yml - Ansible Playbook to join worker nodes with master node.

clear_k8s_setup.yml - Ansible Playbook helps to delete entire configurations from all nodes.

playbooks - Its a directory holds all playbooks.
