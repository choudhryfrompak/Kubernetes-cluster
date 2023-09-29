
# Kubernetes-cluster
This repository is the documentation for clustering multiple machines together as a cluster for containerizing workloads. we will also be installing rancher for a proper interactive interface.
# 


## Documentation

## Pre-requisites:

- At least 2 computer to make cluster (1 master & 1 worker)

- Ubuntu 20.04 installed on all computers (Ubuntu-server on all worker nodes & Ubuntu-Graphical on master node)

- All computers must be on same network

## Initial setup:

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
