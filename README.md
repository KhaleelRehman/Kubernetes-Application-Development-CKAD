# Kubernetes-Application-Development-CKAD
Course material for the CKAD

# Setting up Lab


# Steps to install the Kubeadm
1. you must have multiple system or virtual machines created for configuring a cluster, we will setup your setup to do: 
setting up one as a master and others as worker node

2. intall container runtime on the hosts, as we can run container on the hosts, we will be using docker so e must install docker on all the nodes

3. install kubeadm tools on all the nodes, by insalling and configuring all the components in right nodes

4. initialize the master server, during this process all the required components are installed and configured, we can start the cluster 
level configuration from the master server

5. Pod network

6. join worker node to the master node to launch our application in kubernetes



# 1. Setup virtual machines: 
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/ 

i. install VM https://www.virtualbox.org/wiki/Downloads

ii. download ubuntu image https://www.osboxes.org/virtualbox-images

iii.download and install Mobaxterm https://mobaxterm.mobatek.net/download-home-edition.html

 # create virtual machine using ubuntu image master node
 open oracle Virtual box and click on new to add new VM
 
	name it as kube-master
	version ubuntu-64
	memory must be 2gb or more
	use existing virtual hard disk file - select the path of ubuntu vdi image file
	after creating master vm
	before power it on - open settings of master vm
	goto - system > processor >set up 2 cpus or more
	goto to network > change default address Nat to Bridged Adapter
	now take a snapshot named it Before Initial PowerUp

Now start virtual machine

after that, login will require a password which will be 

    osboxes.org

search terminal - open it

write command to see the ip/inet address assign to it

    ip -br a
# if not getting ipv4 follow the steps otherwise skip this steps
		sudo systemctl enable systemd-networkd.service
		sudo systemctl start systemd-networkd.service
		sudo systemctl status systemd-networkd.service
		sudo dhclient -v enp0s3


now see the ssh service is running

    sudo systemctl status ssh.service


if not running than install it by going to root:

		sudo su
		password: osboxes.org
		apt-get update
		sudo apt-get install openssh-server
		sudo systemctl status ssh.service

if service still not running
		
		sudo apt-get --reinstall install openssh-server


# to ssh terminal open mobaxterm

click on ssh
 remote host: enter the inet address of the master VM, check the specify username and enter
 
     osboxes

press OK
	enter password
  
      osboxes.org
now shut it down

      shutdown now

# Now again go to the oracle VM box
i. right click on kube-master Vmachine

ii. click clone

iii. name it kube-node1 - reinitialize the mac address of each network card - linked clone

right click on kube-master Vmachine again to make second node
click clone
name it kube-node2 - reinitialize the mac address of each network card - linked clone

# power on the all machines check the ipaddress by running the terminal followed by 'ifconfig' command and create ssh as above create for the master node

Next step to set the hostnames
first become root user: 

    sudo su
    
two steps to change the hostname	
    
    cat /etc/hostname

osboxes

    vi /etc/hostname

delete existing name and change to kubemaster - to save - :wq!

    vi /etc/hosts
    
change to kubemaster and save it- :wq!
    
    shutdown now

for other nodes do the same process kubenode1 and kubenode2 - and shutdown

# change static ip address 
goto global tools in the virtual machine

default virual adpater to create if not available and dhcp disabled
connect this adapter to the our VM : setting > network > adapter 2> attached to: host-only adapter
do for both other machines


turn on the machines
run terminal - ifconfig - new interface

does not ip address assigned

    sudo su

set ipaddress: 

    ifconfig enp0s8 192.168.56.2

this is temporary after rebooting

to do permanent:

    vi /etc/network/interfaces

    auto lo
    iface lo inet loopback

     #configure enp0s8 interface
      auto enp0s8
      iface enp0s8 inet static
	        address 192.168.56.2
	        netmask 255.255.255.0
save :wq!


# reboot


for both kubenode1,kubenode2, repeat the steps with 192.168.56.3 and 192.168.56.4

check working
    
     kubemaster ping www.google.com



# swap disable: mobaxterm
    
     swapoff -a
     
     vi /etc/fstab
 
 comnt last line to disable swap


end of step 1

# 2. configure cluster with kubeadm

intall docker https://docs.docker.com/engine/install/ubuntu/#prerequisites


set up repos

run on all the 3 nodes

    sudo apt-get update
    sudo apt-get install \
    ca-certificates \
    curl \
    gnupg


    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    echo \
    "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


# install docker

  
# installing docker, kubectl, kubelet, kubeadm

		sudo apt-get update
		sudo apt-get install docker.io
		sudo apt-get update
		sudo apt-get install -y apt-transport-https curl
		curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
		echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
		sudo apt-get update
		sudo apt-get install -y kubelet kubeadm kubectl

# initialize master node

		sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# installing pod net work
		kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
# if not working alternative pod networks:
flannel

		kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
weave net

		kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
cilium
		
		kubectl create -f https://raw.githubusercontent.com/cilium/cilium/v1.9/install/kubernetes/quick-install.yaml

# join worker node

		sudo kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash <hash>



# check kube-dns running or not - 

    kubectl get pods --all-namespaces

# join nodes paste token

    kubectl get nodes

    kubectl run nginx --image=nginx

    kubectl get pods

    kubectl delete deployment/nginx


# CKAD focuses
1. Core concepts: be able to create and modify basic components, such as pods, services, and deployments.
2. Multi-container pods: Understand how to define and run multiple containers in a pod
3. Configuration: Know how to use ConfigMaps and Secrets to configure applications 
4. Observability: Be familiar with logging and monitoring Kubernetes applications using tools such as kubectl, logs, and metrics.
5. Pod design: Be able to design and create effective and efficient pod configurations using labels, annotations, and affinity/anti-affinity rules.
6. Services and networking: Understand how to create and use services to expose applications and connect pods together within a cluster.
7. State persistence: Know how to manage stateful applications in Kubernetes using StatefulSets, Persistent Volumes, and Persistent Volume Claims.
8. Application lifecycle management: Be able to roll out and roll back deployments, perform rolling updates, and use readiness and liveness probes to monitor application health.

	# Pods

		kubectl apply -f nginx-pod.yaml
		kubectl get pods
		
	# Replicasets
		
		kubectl apply -f nginx-replicaset.yaml
		kubectl get replicasets
		
	# Deployments
		
		kubectl apply -f nginx-deployment.yaml
		kubectl get deployments
		kubectl scale --replicas=5 deployment/nginx-deployment
		kubectl get deployments
	
	# Namespaces
	
		kubectl apply -f my-namespace.yaml
		kubectl get namespaces
		kubectl describe namespace my-namespace

	# Services
	
		kubectl apply -f nginx-service.yaml
		kubectl get services
	
	# Configmaps & Secrets
	
		kubectl apply -f app-config.yaml
		kubectl get configmaps
		kubectl describe configmap <configmap-name>
		kubectl apply -f my-secret.yaml
		kubectl get secrets
		kubectl describe secret <secret-name>

	# Multicontainers

		kubectl apply -f multi-container-pod.yaml
		kubectl get pods
		
	# liveness and readiness probes
		
		kubectl apply -f nginx-liveness-readiness.yaml
		kubectl get deployments


	# jobs
			
		kubectl apply -f pi-job.yaml
		kubectl get jobs
		kubectl describe job <job-name>

	# cronjobs

		kubectl apply -f hello-cron.yaml
		kubectl get cronjobs
		kubectl describe cronjob <cronjob-name>


	# Persistence Volume
	
		kubectl apply -f my-pv.yaml
		kubectl get pv
		kubectl describe pv <pv-name>
		
		kubectl apply -f my-pvc.yaml
		kubectl get pvc
		kubectl describe pvc <pvc-name>
		
	
	# Statefulsets
	
		kubectl apply -f my-statefulset.yaml
		kubectl get statefulsets
		kubectl describe statefulset <statefulset-name>
	
	# Ingress
	
		kubectl apply -f my-ingress.yaml
		kubectl get ingresses
		kubectl describe ingress <ingress-name>
	
	# Network policy
	
		kubectl apply -f my-network-policy.yaml
		kubectl get networkpolicies
		kubectl describe networkpolicy <network-policy-name>
	
	# Resource Quota
	
		kubectl apply -f my-resource-quota.yaml
		kubectl get resourcequotas
		kubectl describe resourcequota <resource-quota-name>
	
	# Horizontal Pod Autoscaling (HPA)
	
		kubectl apply -f my-hpa.yaml
		kubectl get hpa
		kubectl describe hpa <hpa-name>
		
	# Service Accounts
	
		kubectl apply -f my-service-account.yaml
		kubectl get serviceaccounts -n my-namespace
		kubectl describe serviceaccount my-service-account -n my-namespace
		kubectl describe secret <name of token>

	resource limit
	
		kubectl apply -f my-pod.yaml
		kubectl get pods -n my-namespace
		kubectl describe pod my-pod -n my-namespace
		
	liveness and readiness probes
		
		kubectl apply -f readiness-liveness-probes.yaml
		kubectl get pods -n my-namespace
		kubectl describe pod readiness-liveness-probes -n my-namespace
		
	container logging
	
		kubectl apply -f my-pod-log.yaml
		kubectl get pods -n my-namespace
		kubectl describe pod my-pod-log -n my-namespace
		kubectl logs my-pod-log -c my-container -n my-namespace
		
		
	pod design
	
		kubectl apply -f pod.yaml
		kubectl get pods -l app=my-updated-app
		kubectl describe pod another-pod

	rollback and rolling updates
	
		kubectl apply -f updated-deployment.yaml
		kubectl apply -f rollback-deployment.yaml
		kubectl get deployment my-deployment
		kubectl get deployment my-deployment
		kubectl rollout status deployment/my-deployment
		kubectl rollout history deployment/my-deployment
		kubectl rollout undo deployment/my-deployment


		
		
		














