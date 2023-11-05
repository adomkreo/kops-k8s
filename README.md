## kops-kubernetes-cluster-configuration
updated

## Setting up Kubernetes (K8s) Cluster on AWS Using KOPS

1.kops is a software use to create production ready k8s cluster in a cloud provider like AWS.

2. kOPS SUPPORTS MULTIPLE CLOUD PROVIDERS

3. Kops compete with managed kubernestes services like EKS, AKS and GKE

4. Kops is cheaper than the others.

5. Kops create production ready K8S.

6. KOPS create resources like: LoadBalancers, ASG, Launch Configuration, woker node Master node (CONTROL PLANE.

7. KOPS is IaaC

#!/bin/bash
## 1) Create Ubuntu EC2 instance in AWS

## 2a) create kops user
``` sh
 sudo su

 cd kops 
 sudo adduser kops
 sudo echo "kops  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/kops
 sudo su - kops
 mkdir kops ```
 ##  2a) install AWSCLI using the apt package manager
  ```sh
 sudo apt install awscli -y (use 2b option)

 ```
 ## or 2b) install AWSCLI using the script below
 ```sh

sudo apt install unzip wget -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo unzip awscliv2.zip
sudo apt install unzip python-is-python3 -y
sudo ./aws/install

aws --version
 ```
## 3) Install kops software on an ubuntu instance by running the commands below:
 	sudo apt install wget -y
 	sudo wget https://github.com/kubernetes/kops/releases/download/v1.28.0/kops-linux-amd64
 	sudo chmod +x kops-linux-amd64
 	sudo mv kops-linux-amd64 /usr/local/bin/kops
 
## 4) Install kubectl kubernetes client if it is not already installed
```sh
 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo chmod +x ./kubectl
 sudo mv ./kubectl /usr/local/bin/kubectl
 kubectl version --client
```
## 5) Create an IAM role from AWS Console or CLI with the below Policies. 

	AmazonEC2FullAccess 
        AmazonRoute53FullAccess
	AmazonS3FullAccess
	IAMFullAccess 
	AmazonVPCFullAccess
        AmazonSQSFullAccess
	AmazonEventBridgeFullAccess

Then Attach IAM role to ubuntu server from Console Select KOPS Server --> Actions --> Instance Settings --> Attach/Replace IAM Role --> Select the role which
You Created. --> Save.
## 6) create a hosted xone
smartuniversaldevops.com (for domain name)
choose private
choose the same region us-east-2a
and chose the same vpc as the kops-server

## 6) create an S3 bucket
## Execute the commands below in your KOPS control Server. use unique s3 bucket name. If you get bucket name exists error.
	aws s3 mb s3://glenburnieahmed
	aws s3 ls # to verify
	
 ## 6b) create an S3 bucket    
	Expose environment variable:
    # Add env variables in bashrc
    
       vi .bashrc
	# Give Unique Name And S3 Bucket which you created.
        export NAME=glenburnieahmed.smartuniversaldevops.com
	export KOPS_STATE_STORE=s3://glenburnieahmed
        
	source .bashrc  
 
 or vi ~/.profile
### 7) Create sshkeys before creating cluster
```sh
    ssh-keygen
 ```

# 8) Create kubernetes cluster definitions on S3 bucket
```sh
Create a cluster in AWS in a single zone. #########this only create the cluster definition
kops create cluster --cloud=aws --zones=us-east-2a --name=glenburnieahmed.smartuniversaldevops.com --dns-zone=smartuniversaldevops.com --dns private
#######################
Suggestions:
 * list clusters with: kops get cluster
 * edit this cluster with: kops edit cluster glenburnieahmed.smartuniversaldevops.com
 * edit your node instance group: kops edit ig --name=glenburnieahmed.smartuniversaldevops.com nodes-us-east-2a
 * edit your control-plane instance group: kops edit ig --name=glenburnieahmed.smartuniversaldevops.com control-plane-us-east-2a
Finally configure your cluster with: kops update cluster --name glenburnieahmed.smartuniversaldevops.com --yes --admin
######################################################

NOW LETS CREATE THE CLUSTER
1- EDIT THE INSTANCE GROUP TO CHANCE TO T2.MEDIUM  ALSO THE CONTROL PLANE TO T2.MEDIUM AND AND LEAVE NUMBER OF INSTANCE (1) AND ALSO THE NUMBER OF NODES
kops edit ig --name=glenburnieahmed.smartuniversaldevops.com control-plane-us-east-2a
2 AFTER THE CHANGES THEN RUN THIS TO CREATE THE CLUSTER 
kops update cluster --name glenburnieahmed.smartuniversaldevops.com --yes --admin


# copy the sshkey into your cluster to be able to access your kubernetes node from the kops server
kops create sshpublickey glenburnieahmed.smartuniversaldevops.com -i /home/kops/.ssh/id_rsa.pub

```
# 9) Initialise your kops kubernetes cluser by running the command below
```sh
kops update cluster ${NAME} --yes
```
# 10a) Validate your cluster(KOPS will take some time to create cluster ,Execute below commond after 3 or 4 mins)

kops validate cluster
	   
	   Suggestions:
 * validate cluster: kops validate cluster --wait 10m
 * list nodes: kubectl get nodes --show-labels
 * ssh to the master: ssh -i ~/.ssh/id_rsa ubuntu@api.class.k8s.local
 * the ubuntu user is specific to Ubuntu. If not using Ubuntu please use the appropriate user based on your OS.
 * read about installing addons at: https://kops.sigs.k8s.io/operations/addons.

## 10b - Export the kubeconfig file to manage your kubernetes cluster from a remote server. For this demo, Our remote server shall be our kops server 
```sh
 kops export kubecfg $NAME --admin
```
## 11a) To list nodes and pod to ensure that you can make calls to the kubernetes apiSAerver and run workloads
	  kubectl get nodes 

### 11b) Alternative you can ssh into your kubernetes master server using the command below and manage your cluster from the master
    sh -i ~/.ssh/id_rsa ubuntu@ipAddress
    ssh -i ~/.ssh/id_rsa ubuntu@18.222.139.125
    ssh -i ~/.ssh/id_rsa ubuntu@172.20.58.124

### 11b. Alternative, Enable PasswordAuthentication in the master server and assign passwd
```sh
sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
sudo service sshd restart
sudo passwd ubuntu
```

### 11c) To list nodes

	  kubectl get nodes 
 
## 12) To Delete Cluster

   kops delete cluster --name=${NAME} --state=${KOPS_STATE_STORE} --yes  
   
====================================================================================================


13 # IF you want to SSH to Kubernetes Master or Nodes Created by KOPS. You can SSH From KOPS_Server

sh -i ~/.ssh/id_rsa ubuntu@ipAddress
ssh -i ~/.ssh/id_rsa ubuntu@3.90.203.23
  
``
