# CyberProject1 -UofM Cybersecurity Bootcamp
GitHub submission repository for Project 1

Ian Stanford 23 December, 2021

---
## Project one -Azure Cloud Environment with Elk Stack Monitoring
### A Cloud environment and security with an Elk Stack monitoiring deployment within the internal network. 
This Repository contains files and a diagram for the Project One Azure cloud-based Elk Stack network environment below.![ProjectOneDiagram.png](https://github.com/IanJStan/CyberProject1/blob/main/Diagrams/ProjectOneDiagram.png)
I was able to successfully create and launch an Elk Stack monitoring server into my Azure network. Many of the files used for YAML playbooks and configurations were pre-constructed for the class to avoid any manual errors and due to length. Each script only needed minor changes with internal IP addresses, naming, and use of ports. 

---
### Topology of the Network
I first constructed an Azure environment in the previous week of class by creating an Azure account. THe first task is to construct a Red Team Resource Group which the entire network will be contained within. The first of two Virtual Networks were created within this resource group. The Red Team Virtual Network encompasses the Red Team network environment to include the firewall, Jump Box Provisioner, the Load Balancer, and the two DVWA web servers. The Elk Net Virtual Network encompasses the Elk server network to include the firewall and the VM containing the Elk monitoring environement. Both networks are *peered* to each other to allows for a seamless communications connection between the two networks. 

Two Network Security Groups (NSG) were established to control traffic through the firewall within each network. The Red Team NSG and the Elk1 NSG. More detail about the inbound security rules I added to each NSG are included below. 

THe RedTeam NSG provides security/ firewall protection to the Jumpbox Provisioner and to the two DVWA [Damn Vulnerable Web Application](https://dvwa.co.uk/) web servers within the Red Team virtual network. The Jumpbox is assessible by my local IP address with asymmetric key sharing between my host computer and the provisioner. Two additional virtual machines are created within the same availability set to act as DVWA servers. These two servers only maintain internal accessible IP networking addresses and were tested for redunancy after set-up by taking each webserver off-line individually to be sure that this did not affect the DVWA access. 

A Docker container was installed onto the Jumpbox Provisioner in order to run applications such as the Ansible provisioner. Ansible was built into the Docker container to manage the configuration management in a more steamline fashion. The Ansible configuration files and YAML playbooks, to include Metricbeat and Filebeat files, can be found within this container. New asymmetic security keys were constructed from this container to each of the DVWA web servers. 

A Red Team Load Balancer was put into the network to forward HTTP standard TCP traffic through port 80 to the Red Team virtual network. A backend pool and health probes were added for reducancy, health monitoring, and vunerability protections. 

| Name               | Public IP     | Private IP | Function      | Security Group | Virtual Network |   OS       |
|:------------------:|:-------------:|:----------:|:-------------:|:--------------:|:---------------:|:----------:|
| JumpBoxProvisioner | 40.122.54.192 | 10.0.0.4   | Gateway       | RedTeamNSG     | Red Team        | Linux      |
| Web-1              | 13.67.151.73  | 10.0.0.5   | DVWA          | RedTeamNSG     | Red Team        | Linux      |
| Web-2              | 13.67.151.73  | 10.0.0.6   | DVWA          | RedTeamNSG     | Red Team        | Linux      |
| Red-Team-LB        | 13.67.151.73  |     N/A    | Load Balancer | RedTeamNSG     | Red Team        | Linux      |
| Elk-1		           | 40.117.89.35  | 10.1.0.5   | Monitoring    | Elk1nsg743     | ELK-NET         | Linux      |
| Host               | 72.50.206.6   |     N/A    | Local Host    | All            | All             | Windows 10 |


---
#### Network Security Groups and Firewalls
Two NSG firewalls were deployed with in the resource group. The RedTeamNSG secured the Red Team virtual network using three manual Inbound Security Rules along with built-in rules:
| Priority  |  Name          | Port  | Protocol | Source       | Destination     | Service | Function                  |
|:---------:|:--------------:|:-----:|:--------:|:------------:|:---------------:|:-------:|--------------------------:|
| 499       | Access to Vnet | 80    | TCP      | 72.50.206.6  | Virtual Network | HTTP    | Access to Virtual Network |
| 500       | AllowSSH       | 22    | TCP      | 72.50.206.6  | Virtual Network | SSH     | Allow access from my IP   |
| 501       | JumpBox-Access | 22    | TCP      | 10.0.0.4     | Virtual Network | SSH     | SSH access from Jump Box  |

The Elk1nsg743 secured the ELK-NET virtual network using two Inbound Security Rules along with built-in rules:
| Priority  |  Name          | Port  | Protocol | Source       | Destination     | Service | Function                  |
|:---------:|:--------------:|:-----:|:--------:|:------------:|:---------------:|:-------:|--------------------------:|
| 200       | Elk5601TCP     | 5601  | TCP      | 72.50.206.6  | Virtual Network | Custom  | Elk port 5601 TCP         |
| 300       | SSH            | 22    | TCP      |     Any      |      Any        | SSH     | Port 22 to Internet       |

---
#### ELK Stack Installation
Within the Elk Net Virtual Network another VM was created to act as a server for the Elk Stack monitoring applications to include:
1. Elasticsearch
2. Logstack
3. Kibana
4. Beats using Metricbeat and Filebeat.

From the JumpBox Provisioner Ansible container, asymmetric keys were created to configure the new ELK VM. At this point, the Elk Server is sharing the same Public Keys as the two DVWA webservers from the Ansible container. An Elk-Install playbook was run to configure ELK VM with Docker. This playbook installed:
* docker.io
* python3-pip
* docker, which is the Docker Python pip module
This ELK server is running on port 5601. With [Kibana](https://elestic.com/kibana/kibana-dashboard/) monitoring installed and working through http://40.117.89.35:5601/app/kibana (this ELK server is not currently running), Beats were added through Elastic using Filebeat and Metricbeat. This allows for monitoring of the webservers for vulnerabilities that may appear in system logs and data. ELK enables the ability to search, analyze, and store monitoring activity, and to present it visually for analysis. 
Kibana monitoring from Elk Stack server. Below is a sample screenshot of Kibana from my Elk server:
![Kibana screenshot](https://user-images.githubusercontent.com/96362831/147275913-ed9b83a1-bd2c-4d13-a906-9218d0a94273.png)
