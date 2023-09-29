
# Kubernetes-cluster
This repository is the documentation for clustering multiple machines together as a cluster for containerizing workloads. we will also be installing rancher for a proper interactive interface.
# 


## Documentation

## Pre-requisites:

- At least 2 computer to make cluster (1 master & 1 worker)

- Ubuntu 20.04 installed on all computers (Ubuntu-server on all worker nodes & Ubuntu-Graphical on master node)

- All computers must be on same network

## Initial setup:
## Enabling SSH
- after installing  OS on all the computers. you have to enable ssh access on all computers. by following the processs below >
 -open the terminal and run:

```bash
  sudo apt-get update
```
let the packages update. after that run:

```bash
  sudo apt install openssh-server
```
This command will install openssh-server on your machine

Enable ssh server on Ubuntu, run: 
```bash
  sudo systemctl enable ssh
```
By default, firewall will block ssh access. Therefore, you must enable ufw and open ssh port
Open ssh tcp port 22 using ufw firewall, run:

```bash
  sudo ufw allow ssh
```
Now to check status of ssh-server run:
```bash
    sudo systemctl status ssh
```
Your ssh is now enabled:

- Now to get your public ip adress run:

```bash
    hostname -I
```
note your IP ADDRESS and machine name and make a command like:

```bash
    ssh <username>@<ip-address>
```
Replace <username> with your machine name and <ip-address> with your machine ip. the command will look like:

```bash
    ssh master@192.168.10.2
```
# REPEAT THAT PROCESS ON ALL MASTER AND WORKER NODES
you can run this command on your laptop powershell terminal to access and configure your master and worker nodes.

- Now all the nodes are accesible through ssh so we can start our cluster setup : )

## Master Node Configurations:

- ssh into the master node by using the command you created for the master node that contain its name and ip address.

```bash
    ssh <username>@<ip-address>
```
enter the password and login to access. and run this command for root accesss

```bash
    sudo su -
```
entering your password and you will get root access.

- now to proceed you need to install curl to pull packages from the internet. to install curl run:

```bash
    sudo apt install curl
```
after installing curl pull the K3s cluster image from internet by running:

## This process is for master node:

Run this command on master node.
```bashrc
    curl -sfL https://get.k3s.io |  K3S_KUBECONFIG_MODE="644" sh -s
```

- this will pull image and run it on master node. and i am also adding a configuration in it to make it able to run on rancher.

after doing that we have ```kubectl``` available as a command that will help us do everything.
- to confirm that the masterr node is properly configured

run:
```bashrc
kubectl get nodes
```

That is it the master node is now configured. Now to setup the worker node we need a token from our master node.
to get that token. Run:

```bashrc
    cat var/lib/rancher/k3s/server/node-token
```
Running this command will show the token, copy and save that token.
Now we are gonna create a configuration command for our worker node that will be like:

```bashrc
    curl -sfL https://get.k3s.io | K3S_TOKEN="YOUR_TOKEN" K3S_URL="https://[ip of master]:6443" K3S_NODE_NAME="workernodename" sh -s
```

- here replace ```[ip of master]``` with the master node ip address.
- Replace ```YOUR_TOKEN``` wwith the token we cat & saved from master node recently.
- Replace ``workernodename``` with your desire name for your worker node to be registered inside master.

Once your command is ready. you need to ssh into your worker node. by using ```ssh username@ip_address``` inside a new powershell terminal. Login into the worker node and enabke the root access by running 

```
   sudo su -

```

After getting the root access paste the command you make from this template ```curl -sfL https://get.k3s.io | K3S_TOKEN="YOUR_TOKEN" K3S_URL="https://[ip of master]:6443" K3S_NODE_NAME="workernodename" sh -s```

Run this command.

# REPEAT THIS PROCESS FOR ALL WORKER NODES.

Now go back to your MASTER NODE and run this command to check whether the worker nodes are registered or not.

Run:
```bashrc
    kubectl get nodes
```
THis will show you Master and all the worker nodes that are registered.

Congratulation the Kubernetes cluster has now been successfully configured by this Guided processs.

## Setting Up Rancher
- No doubt that our cluster is now properly usable and ready for deployment of Pods. But

- For a proper Graphical Dashboard monitoring and management system we are installing rancher.

- The whole proccess of installiing rancher will be carried on the master node on which we have installed Graphical version of ubuntu 20.04. 

- We need to create a dedicated virtual machine to run rancher on cluster. we will import our cluster to that server.

- connect a monitor to the machine on which you set up masteer node and open firefox.

## Download virtualbox

- you need to download virtualbox to create a virtual machine inside master node computer.
- Also download ubuntu server iso file to install in virtual machine
- you may encounter network ip related problems later so to expose it to local network configure the network setting of the virtualbox vm to ```NAT```.
- Virtual machine configuration should be at least 
```bashrc
    4 cores 
    4gb ram
    40gb disk space
```
Once you have initiated the VM configure its `SSH ACCES` like we did on other machines in start and get its `IP ADDRESS`

Access this VM fromany other computer lke your laptop powershell to check whetherit is publically available or not by using `ssh username@ip_address`

After accessing it we can now start setting up rancher.

## installing docker-engine
- we need  to install docker image to containerize rancher image

- we can INSTALL the compaitable version suggested by rancher by running this command 

```bashrc
    curl https://releases.rancher.com/install-docker/20.10.sh | sh
```

- Note that the following sysctl setting must be applied:

```net.bridge.bridge-nf-call-iptables=1```

to confirm whether docker is properly installed run ```docker``` 

## containerizing rancher image:
we need to make rancher data presit accross reboots.
- Rancher by default stores its data in ```/var/lib/rancher```
Lets create a space to save data into our specified directory.
we will make one by Running:
```mkdir /kubeernetes/rancher/volume```

Now as the docker is properly installed and our volume folder is ready we can proceed by running a command that i have set up to make the installation easy.

```cd /kubeernetes/rancher``` 

Run:
```bashrc
docker run -d --name rancher-server -v ${PWD}/volume:/var/lib/rancher --restart=unless-stopped -p 80:80 -p 443:443 --privileged rancher/rancher:latest
```
- After running this wait for few minutes to let the rancher serveer start

Then go to a browser and type the ip address of your virtual machine that is spinning up on virtualbox with the port 443 like this:
```bashrc
https://<ip_address>:443
```
## Unlocking Rancher Dashboard

By default the rancher dashboard is locked the default password is somewhere in the logs of docker container and we need to pull it out for that we will go through some commands
-Go back to terminal of virtual box vm
we will need the container id on which rancher is spinning up. to get that, run: 
```bashrc
docker ps

```

- Now we got the container id run this command:
```bashrc
docker logs <container-id> 2>&1 |grep "Bootstrap Password:"
```
- Replace the ```<container-id>``` with the id of your container. when you run this command this will grab the bootstrap password.

- Now again open the browser and enter that password 
- Now it will ask you to change the password. set your desired password & proceed
Rancher is now properly installed on your virtulbox.

the next step is to import our main cluster into it.

## importing main cluster to Rancher

As you are now on the homepage of rancher dashboard you will see a icon on top right that says `import existing` click on that and name your cluster and proceed. no need to mess with any other setting.

Now it wwill show you some configuration command that starts with `curl -sfl` we will need the command with `-insecure` tag because wwe dont have ssl certificate don't worry we really dont need that the command i am talking about maybe it is last among those three commands. 
- copy that command and open terminal of your master node and don't forget enabling root accesspaste and run. wait for its completion.

- Go back to browser , you will see that your cluster is succesfully added showing all the resources and ready for containerizing pods. 
## Happy Clustering :)



## Author

- [CHOUDHRY](https://www.github.com/choudhryfrompak22)

