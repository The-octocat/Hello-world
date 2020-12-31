![CI](https://github.com/github/visualstudio/workflows/CI/badge.svg)

[![Follow GitHub for Visual Studio](https://img.shields.io/twitter/follow/GitHubVS.svg?style=social "Follow GitHubVS")](https://twitter.com/githubvs?ref_src=twsrc%5Etfw) [![Join the chat at https://gitter.im/github/VisualStudio](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/github/VisualStudio?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
![Azure Public Test Date](https://azurequickstartsservice.blob.core.windows.net/badges/101-aci-storage-file-share/PublicLastTestDate.svg)
![Azure Public Test Result](https://azurequickstartsservice.blob.core.windows.net/badges/101-aci-storage-file-share/PublicDeployment.svg)

![Azure US Gov Last Test Date](https://azurequickstartsservice.blob.core.windows.net/badges/101-aci-storage-file-share/FairfaxLastTestDate.svg)
![Azure US Gov Last Test Result](https://azurequickstartsservice.blob.core.windows.net/badges/101-aci-storage-file-share/FairfaxDeployment.svg)

![Best Practice Check](https://azurequickstartsservice.blob.core.windows.net/badges/101-aci-storage-file-share/BestPracticeResult.svg)
![Cred Scan Check](https://azurequickstartsservice.blob.core.windows.net/badges/101-aci-storage-file-share/CredScanResult.svg)

[![Deploy To Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-aci-storage-file-share%2Fazuredeploy.json)
[![Deploy To Azure US Gov](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazuregov.svg?sanitize=true)](https://portal.azure.us/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-aci-storage-file-share%2Fazuredeploy.json)
[![Visualize](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/visualizebutton.svg?sanitize=true)](http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-aci-storage-file-share%2Fazuredeploy.json)
# Clusters with Swarm Orchestrators

These Microsoft Azure Stack templates create docker Swarm clusters.

The below content is to give overall architecture of the Swarm Cluster

## Deployed resources

 Once your cluster has been created you will have a resource group containing 2 parts:

 1. a set of 1,3,5 masters nodes.  Each master's SSH can be accessed via the public dns address at ports 2200..2204

 2. a set of agents nodes.  The agent VMs must be accessed through the master.

  The following image is an example of a cluster with 3 masters, and 3 agents:

 ![Image of Swarm cluster on azure](https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/develop/docker-swarm-ubuntu/images/swarm.png)

 All VMs are on the same private subnet, 10.0.0.0/18, and fully accessible to each other.

## Prerequisites

Follow the below links to create/download an Ubuntu 14.04 LTS Image and upload the same to Azure Stack's Platform Image Repository(PIR)
1. https://azure.microsoft.com/en-us/documentation/articles/azure-stack-linux/
2. https://azure.microsoft.com/en-us/documentation/articles/azure-stack-add-image-pir/
	Note: please use the default values for linuxPublisher,linuxOffer,linuxSku,linuxVersion found in azuredeploy.json while creating the manifest.json in PIR

## Deployment steps
=> Deploy to azurestack, using custom deployment in azurestack portal.
=> or use Deployswarm.ps1 to deploy to azurestack via powershell.

## Usage
1. Get your endpoints to cluster
 1. browse to https://portal.azurestack.local

 2. then click browse all, followed by "resource groups", and choose your resource group

 ![Image of resource groups in portal](https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/develop/docker-swarm-ubuntu/images/portal-resourcegroups.png)

 3. then expand your resources, and copy the dns names of your jumpbox (if chosen), and your NAT public ip addresses.

 ![Image of public ip addresses in portal](https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/develop/docker-swarm-ubuntu/images/portal-publicipaddresses.png)

2. Connect to your cluster using windows jumpbox - remote desktop to the windows jumpbox 

## Explore Swarm with Simple hello world
 1. After successfully deploying the template write down the two output master and agent FQDNs.
 2. SSH to port 22 of the master private ip
 3. Type `docker -H 10.0.0.5:2375 info` to see the status of the agent nodes.
 ![Image of docker info](https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/develop/docker-swarm-ubuntu/images/dockerinfo.png)
 4. Type `docker -H 10.0.0.5:2375 run hello-world` to see the hello-world test app run on one of the agents

## Explore Swarm with a web-based Compose Script, then scale the script to all agents
 1. After successfully deploying the template write down the two output master and agent FQDNs.
 2. create the following docker-compose.yml file with the following content:
```
web:
  image: "yeasy/simple-web"
  ports:
    - "80:80"
  restart: "always"
```
 3.  type `export DOCKER_HOST=10.0.0.5:2375` so that docker-compose automatically hits the swarm endpoints
 4. type `docker-compose up -d` to create the simple web server.  this will take about a minute to pull the image
 5. once completed, type `docker ps` to see the running image.
 ![Image of docker ps](https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/develop/docker-swarm-ubuntu/images/dockerps.png)
 6. You can now scale the web application by typing `docker-compose scale web=3`, and this will scale to the rest of your agents.  The Azure load balancer will automatically pick up the new containers.
 ![Image of docker scaling](https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/develop/docker-swarm-ubuntu/images/dockercomposescale.png)

 ## Notes
 * This version of swarm is a non-HA(no Loadbalancer or Availabilitysets) with master, Agent node deployment.
 * Refer https://help.ubuntu.com/community/SSH/OpenSSH/Keys#Generating_RSA_Keys for generating sshkeys for ubuntu
 * the installation log for the linux jumpbox, masters, and agents are in /var/log/azure/cluster-bootstrap.log
 * event though the VMs finish quickly configuring swarm can take 5-15 minutes to install, check /var/log/azure/cluster-bootstrap.log for the completion status.


# Deploy Two VM Extensions to existing Windows VM (Custom Script with command and BGInfo)

This template deploys both the Custom Script and BGInfo VM Extensions to an existing Windows VM in an Azure Stack environment. It takes a parameter for vmName, and allows for the execution of a command (as seen in the commandToExecute Extension setting).
