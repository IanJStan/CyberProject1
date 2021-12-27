# CyberProject1 -UofM Cybersecurity Bootcamp
GitHub submission repository for Project 1

Ian Stanford 23 December, 2021

---
## Project one -Azure Cloud Environment with Elk Stack Monitoring
### A Cloud environment and security with an Elk Stack monitoiring deployment within the internal network. 
The files in this repository were used to configure the network depicted below.![ProjectOneDiagram.png](https://github.com/IanJStan/CyberProject1/blob/main/Diagrams/ProjectOneDiagram.png)
These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the YAML file may be used to install only certain pieces of it, such as Filebeat.
I was able to successfully create and launch an Elk Stack monitoring server into my Azure network. Many of the files used for YAML playbooks and configurations were pre-constructed for the class to avoid any manual errors and due to length. Each script only needed minor changes with internal IP addresses, naming, and use of ports. 
This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


---
### Topology of the Network
The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.
I first constructed an Azure environment in the previous week of class by creating an Azure account. THe first task is to construct a Red Team Resource Group which the entire network will be contained within. The first of two Virtual Networks were created within this resource group. The Red Team Virtual Network encompasses the Red Team network environment to include the firewall, Jump Box Provisioner, the Load Balancer, and the two DVWA web servers. The Elk Net Virtual Network encompasses the Elk server network to include the firewall and the VM containing the Elk monitoring environement. Both networks are *peered* to each other to allows for a seamless communications connection between the two networks.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network. A Red Team Load Balancer was put into the network to forward HTTP standard TCP traffic through port 80 to the Red Team virtual network. A backend pool and health probes were added for reducancy, health monitoring, and vunerability protections.

Two Network Security Groups (NSG) were established to control traffic through the firewall within each network. The Red Team NSG and the Elk1 NSG. More detail about the inbound security rules I added to each NSG are included below. 

THe RedTeam NSG provides security/ firewall protection to the Jumpbox Provisioner and to the two DVWA [Damn Vulnerable Web Application](https://dvwa.co.uk/) web servers within the Red Team virtual network. The Jumpbox is assessible by my local IP address with asymmetric key sharing between my host computer and the provisioner. Two additional virtual machines are created within the same availability set to act as DVWA servers. These two servers only maintain internal accessible IP networking addresses and were tested for redunancy after set-up by taking each webserver off-line individually to be sure that this did not affect the DVWA access. 

A Docker container was installed onto the Jumpbox Provisioner in order to run applications such as the Ansible provisioner. Ansible was built into the Docker container to manage the configuration management in a more steamline fashion. The Ansible configuration files and YAML playbooks, to include Metricbeat and Filebeat files, can be found within this container. New asymmetic security keys were constructed from this container to each of the DVWA web servers. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the logs and system performance. Filebeats monitors the generated log files while Metricbeat records metric data along with the operating system functions to include memory usage and CPU's based on the different services as they are applied. 
 

The configuration details of each machine may be found below.
| Name               | Public IP     | Private IP | Function      | Security Group | Virtual Network |   OS       | Public Access  |
|:------------------:|:-------------:|:----------:|:-------------:|:--------------:|:---------------:|:----------:|:--------------:|
| JumpBoxProvisioner | 40.122.54.192 | 10.0.0.4   | Gateway       | RedTeamNSG     | Red Team        | Linux      |       NO       |
| Web-1              | 13.67.151.73  | 10.0.0.5   | DVWA          | RedTeamNSG     | Red Team        | Linux      |       NO       |
| Web-2              | 13.67.151.73  | 10.0.0.6   | DVWA          | RedTeamNSG     | Red Team        | Linux      |       NO       |
| Red-Team-LB        | 13.67.151.73  |     N/A    | Load Balancer | RedTeamNSG     | Red Team        | Linux      |       YES      |
| Elk-1		           | 40.117.89.35  | 10.1.0.5   | Monitoring    | Elk1nsg743     | ELK-NET         | Linux      |       NO       |
| Host               | My IP         |     N/A    | Local Host    | All            | All             | Windows 10 |       NO       |


---
### Access Policies
The machines on the internal network are not exposed to the public Internet.
Only the JumpBox Provisioner machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
My IP Address or Local Host IP address.
Machines within the network can only be accessed by Port 22. The JumpBox Provisioner can access the Elk Server via it's internal IP address of 10.0.0.4.
Two NSG firewalls were deployed with in the resource group. The RedTeamNSG secured the Red Team virtual network using three manual Inbound Security Rules along with built-in rules:
| Priority  |  Name          | Port  | Protocol | Source       | Destination     | Service | Function                  |
|:---------:|:--------------:|:-----:|:--------:|:------------:|:---------------:|:-------:|--------------------------:|
| 499       | Access to Vnet | 80    | TCP      |    My IP     | Virtual Network | HTTP    | Access to Virtual Network |
| 500       | AllowSSH       | 22    | TCP      |    My IP     | Virtual Network | SSH     | Allow access from my IP   |
| 501       | JumpBox-Access | 22    | TCP      | 10.0.0.4     | Virtual Network | SSH     | SSH access from Jump Box  |

The Elk1nsg743 secured the ELK-NET virtual network using two Inbound Security Rules along with built-in rules:
| Priority  |  Name          | Port  | Protocol | Source       | Destination     | Service | Function                  |
|:---------:|:--------------:|:-----:|:--------:|:------------:|:---------------:|:-------:|--------------------------:|
| 200       | Elk5601TCP     | 5601  | TCP      |    My IP     | Virtual Network | Custom  | Elk port 5601 TCP         |
| 300       | SSH            | 22    | TCP      |     Any      |      Any        | SSH     | Port 22 to Internet       |

---
### ELK Configuration
Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because it can be run from your own command line and you get a consistent implementation of the scripts regardless of where they are run from. 
The playbook implements the following tasks:
1. Elasticsearch
2. Logstack
3. Kibana
4. Beats using Metricbeat.
5. Beats using Filebeat. 

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![docker ps output](Images/docker_ps_output.png)

From the JumpBox Provisioner Ansible container, asymmetric keys were created to configure the new ELK VM. At this point, the Elk Server is sharing the same Public Keys as the two DVWA webservers from the Ansible container. An Elk-Install playbook was run to configure ELK VM with Docker. This playbook installed:
* docker.io
* python3-pip
* docker, which is the Docker Python pip module
Beats were added through Elastic using Filebeat and Metricbeat. This allows for monitoring of the webservers for vulnerabilities that may appear in system logs and data. ELK enables the ability to search, analyze, and store monitoring activity, and to present it visually for analysis. 
Kibana monitoring from Elk Stack server. Below is a sample screenshot of Kibana from my Elk server:

#### Target Machines and Beats
This ELK server is configured to monitor the following machines:
* Web-1 10.0.0.5
* Web-2 10.0.0.6

We have installed the following Beats on these machines: Metricbeat and Filebeat 7.4.0-amd64.deb. 
These Beats allow us to collect the following information from each machine:
* As mentioned before, we are able to capture, store and dispense files to Kibana for final analysis. *
*  Metricbeat measures operating functions such as CPU usage and data related to operations of applications. 

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the filebeat-config.yml file to Elk-1 VM.
- Update the hosts file to include webservers 10.0.0.5, 10.0.0.6, and 10.1.0.5.
- Run the playbook, and navigate to Kibana to check that the installation worked as expected.
- Which URL do you navigate to in order to check that the ELK server is running?
- * This ELK server is running on port 5601. With [Kibana](https://elestic.com/kibana/kibana-dashboard/) monitoring installed and working through http://40.117.89.35:5601/app/kibana (this ELK server is not currently running)
- ![Kibana screenshot](https://user-images.githubusercontent.com/96362831/147275913-ed9b83a1-bd2c-4d13-a906-9218d0a94273.png)

- Files can be found in the vigorous_jackson container within the folder /etc/ansible. Update the filebeat playbook from the Output line to output from the approriate internal IP address. The Elk server is built from the Jumpbox Provisioner while Filebeat is installed with the Ansible container within the Elk-1 server VM. 
