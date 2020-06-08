# Azure-Cloud-Network-Desgin

This document contains the following details:
- [Description of the Topology](#description-of-the-topology)
- [Access Policies](#access-policies)
- [ELK Configuration](#elk-configuration)
  - [Beats in Use](beats-in-use)
  - [Machines Being Monitored](machines-being-monitored)
- [How to Use the Ansible Build](how-to-use-the-ansible-build)

The files in this repository were used to create and configure the network depicted below.

### Description of the Topology

[Azure-Network-architecture](Images/Azure-Network-architecture.png)

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available (responsive), in addition to restricting traffic to the network.
- The load balancers defend an organization against distributed denial-of-service (DDoS) attack by rerouting heavy traffic from one server to another to eliminate single points of failure, reduce the attack surface, and make it harder to exhaust resources and saturate links. The advantage of a jump box here is to prevents all Azure VM's to expose to the public, which means that the jump box will be our entry point connecting via Remote Desktop Protocol (RDP) from our on-premise network. In addition, the jump box also helps us to open only one port instead of several ports to connect different VMs in the Azure cloud.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the infrastructure and system logs.
- Filebeat is used for collecting and shipping log files. It can be installed on almost any operating system, including as a Docker container.
- Metricbeat collects ships various system-level metrics for various systems and platforms.

The configuration details of each machine may be found below.

| Name     | Function     | IP Address | Operating System |
|----------|--------------|------------|------------------|
| Jump Box | Gateway      | 10.0.0.4   | Linxu            |
| DVWA-VM1 | Webserver    | 10.0.0.5   | Linux            |
| DVWA-VM2 | Webserver    | 10.0.0.6   | Linux            |
| DVWA-VM3 | ELK Server   | 10.1.0.4   | Linux            |
| DVWA-VM4 | Spare Server | 10.1.0.5   | Linux            |

These files have been
- Tested and configure the **jump box** to run **Docker containers** and to install a **container**
- Tested and used to generate **webserver containers**, a live **ELK deployment** and **loadbalancer** on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the yml file may be used to install only certain pieces of it, such as __Filebeat__.

### Configure jump box
- start by installing **docker.io** on the Jump box
        sudo apt install docker.io
        
- verify that the Docker service is running
        sudo systemctl service docker
        
- Once Docker is installed, pull the container **cyberxsecurity/ansible**
        sudo docker pull cyberxsecurity/ansible
    - _NOTE: you can also switch to the root user so you don't have to keep tying **sudo**_
    
- Launch the Ansible container and connect to it using the appropriate Docker commands
  - To start the container
        sudo docker run -ti cyberxsecurity/ansible:latest bash
        
  - To quit the container
        exit

- Create a new security group rule that allows your jump box machine full access to your VNet
  - Get the private IP address of your jump box
  - Go to your security group settings and create an **inbound** rule. Create rules allowing **SSH** connections from your IP address
    - Source: Use the **IP Addresses** setting with your jump box's internal IP address in the field
    - Source port ranges: **Any** or \* can be listed here
    - Destination: Set to **VirtualNetwork**
    - Destination port ranges: Only allow SSH. So, only port **22**
    - Protocol: Set to **Any** or **TCP**
    - Action: Set to **Allow** traffic from your jump box
    - Priority: Priority must be a lower number than your rule to deny all traffic
    - Name: Name this rule anything you like, but it should describe the rule. _For example: SSH from Jump Box_
    - Description: Write a short description similar to: "_Allow SSH from the jump box IP_"
    
### Automated Webserver containers Deployment

To create an Ansible plyabook that installed Docker and configure a VM with the DVWA web app.

#### Steps
- Connect to your jump box
  - ssh -i path/private-key username@Jump-box-VM-public-IP
  
- Connect to the Ansible container in the box
  - if you don't remember the name of your container, run the command below to check the name
        sudo docker container list -a
        
  - Start the container using
        sudo docker start container_name
        
  - Get a shell in your container using
        sudo docker attach container_name
        
- Create a YAML playbook file that you will use for web app configuration
  - /etc/ansible/[webserver-playbook.yml](YMAL/webserver-playbook.yml)
    - Use the Ansible **apt** module to install **docker.io** and **python-pip**
    
            - name: docker.io
              apt:
              force_apt_get: yes
              name: docker.io
              state: present
           
            - name: Install pip
              apt:
              force_apt_get: yes
              name: python-pip
              state: present"
              
  - Use the Ansible **pip** module to install **docker**
    - _TODO: script here
    
  - Use the Ansible **docker-container** module to install the **cyberxsecurity/dvwa** container (make sure you publish port **80** on the container to port **80** on the host
    - _TODO_: script here
    
  - Run your Ansible playbook on the new virtual machine
        sudo ansible-playbook /etc/ansible/webserver-playbook.yml
        
  - To test that DVWA is running on the new VM, **SSH** to the new VM from your Ansible container
    ssh ansible-VM-name@new-VM-IP
    
    - run the following **_curl_** command to test the connection, if everything is working, you should get back some **HTML** from the DVWA container
            curl localhost/setup.php
      
- _TODO: may need more details about how to provision
     
### Setup Load Balancer

  - _TODO: steps here
 
### Redundancy Testing

- _TODO:  more details here

### Automated ELK Stack Deployment

  - [elk-playbook.yml](YMAL/elk-playbook.yml)
  - [filebeat-playbook.yml](YMAL/filebeat-playbook.yml)
  - [filebeat-configuration.yml](YMAL/filebeat-configuration.yml)
  - [filebeat.yml](YMAL/filebeat.yml)






### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump-box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- _Add whitelisted IP addresses into the firewall rules to allow the inbound access, ex. I added the IP address of my local machine_

Machines within the network can only be accessed by the Jump-box machine and other machines in the network.
- Only my local machine which is in the whitelisted machine added into the firewall rules (you can use [https://whatismyipaddress.com/](https://whatismyipaddress.com/) to check your IP address) from Internet can access the ELK server besides all other machines within the network.

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses                                                                               |
|----------|---------------------|----------------------------------------------------------------------------------------------------|
| Jump Box | Yes                 | IP-in-whitelist (ex. IP-of-my-own-machine)                                                         |
| DVWA-VM1 | Yes                 | Any                                                                                                |
| DVWA-VM2 | Yes                 | Any                                                                                                |
| DVWA-VM3 | Yes                 | 10.0.0.4 10.0.0.5 10.0.0.6 10.1.0.5 IP-in-whitelist (from external network, ex. My-own-machine-IP) |
| DVWA-VM4 | Yes                 | 10.0.0.4 10.1.0.4                                                                                  |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- Free: Ansible is an open-source tool
- Idempotent: can be used to deploy to the multiple machine with identical settings
- Powerful: Ansible lets you model even highly complex IT workflows
- Efficient: you don't need to install any extra software, there's more room for application resources on your server
- Tiny Learning Curve: no special coding skills are needed to use Ansible's playbook

The playbook implements the following tasks:
- _TODO: In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc._
- ...
- ...

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![TODO: Update the path with the name of your screenshot of docker ps output](Images/docker_ps_output.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- _TODO: List the IP addresses of the machines you are monitoring_

We have installed the following Beats on these machines:
- _TODO: Specify which Beats you successfully installed_

These Beats allow us to collect the following information from each machine:
- _TODO: In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the _____ file to _____.
- Update the _____ file to include...
- Run the playbook, and navigate to ____ to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
- _Which URL do you navigate to in order to check that the ELK server is running?

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
