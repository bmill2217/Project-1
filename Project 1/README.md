## Automated ELK Stack Deployment

The files in this repository were used to configure the network linked below.

![Network Diagram](https://ucsd.bootcampcontent.com/cldearmond/ucsd-cyber-repository-cd/blob/master/Project%201/Diagrams/Kibana_Final_Diagram.jpg)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select playbooks may be used to install needed containers, such as Filebeat.

 ## Docker DVWA install and deployment
  - - name: Config Web VM with Docker
      hosts: webservers
      become: true
      tasks:
      - name: docker.io
        apt:
          force_apt_get: yes
          update_cache: yes
          name: docker.io
          state: present

      - name: Install pip3
        apt:
          force_apt_get: yes
          name: python3-pip
          state: present

      - name: Install Docker python module
        pip:
          name: docker
          state: present

      - name: download and launch a docker web container
        docker_container:
          name: dvwa
          image: cyberxsecurity/dvwa
          state: started
          published_ports: 80:80

      - name: Enable docker service
        systemd:
          name: docker
          enabled: yes - name: Config Web VM with Docker
      hosts: webservers
      become: true
      tasks:
      - name: docker.io
        apt:
          force_apt_get: yes
          update_cache: yes
          name: docker.io
          state: present

      - name: Install pip3
        apt:
          force_apt_get: yes
          name: python3-pip
          state: present

      - name: Install Docker python module
        pip:
          name: docker
          state: present

      - name: download and launch a docker web container
        docker_container:
          name: dvwa
          image: cyberxsecurity/dvwa
          state: started
          published_ports: 80:80

      - name: Enable docker service
        systemd:
          name: docker
          enabled: yes

## Elk Install and Deployment
- hosts: elk
  become: true
  tasks:
    - name: docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

    - sysctl:
       name: vm.max_map_count
       value: '262144'
       state: present

    - name: Install python3-pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

    - name: Install Docker python module
      pip:
        name: docker
        state: present

    - name: download and launch a docker web container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: present
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

    - name: Enable docker service
      systemd:
        name: docker
        enabled: yes

## Install and deploy Filebeat
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O "https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb"

  - name: install filebeat deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: sudo service filebeat start

## Install and deploy Metricbeat
- name: installing and launching Metricbeat
  hosts: webservers
  become: yes
  tasks:

  - name: download Metricbeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb

  - name: install metricbeat deb
    command: dpkg -i metricbeat-7.6.1-amd64.deb

  - name: drop in Metricbeat.yml
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

  - name: enable and configure system module
    command: metricbeat modules enable docker

  - name: setup metricbeat
    command: sudo metricbeat setup

  - name: start metricbeat service
    command: sudo service metricbeat start

##

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
- Load balancers help tp protect against dDos and Dos attacks by ensuring that web assets are deployed across multiple machines. Use of a Jump Box machine allows for ansible configuration deployments across multiple VM's at once using one Playbook. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the DVWA and system resources.
- Filebeat recieves system logs from all VM's hosting web assets.
- Metricbeat monitors system resources across all VM's hosting web assets

The configuration details of each machine may be found below.

| Name      	| Function   	| IP Address 	| Operating System   	|
|-----------	|------------	|------------	|--------------------	|
| Jump Box  	| Gateway    	| 10.0.0.4   	| Linux Ubuntu 18.04 	|
| Web-1     	| DVWA Host  	| 10.0.0.7   	| Linux Ubuntu 18.04 	|
| Web-2     	| DVWA Host  	| 10.0.0.9   	| Linux Ubuntu 18.04 	|
| Web-3     	| DVWA Host  	| 10.0.0.11  	| Linux Ubuntu 18.04 	|
| Elk Stack 	| Elk Server 	| 10.1.0.4   	| Linux Ubuntu 18.04 	|

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 68.7.183.211

Machines within the network can only be accessed by The Jump_Box_Provisioner.
- _The Jump_Box_Provisioner has been granted access to the Elk machine via IP's 67.7.183.211 & 10.0.0.4

A summary of the access policies in place can be found in the table below.

| Name      	| Publicly Accessible 	| Allowed IP Addresses  	|
|-----------	|---------------------	|-----------------------	|
| Jump Box  	| Yes                 	|45.7.183.211          	|
| Web-1     	| No                  	| 10.0.0.4              	|
| Web-2     	| No                  	| 10.0.0.4              	|
| Web-3     	| No                  	| 10.0.0.4              	|
| Elk Stack 	| Yes                 	| 45.7.183.211/10.0.0.4 	|

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because automation saves us time and assures that no steps are missed in the configuration process.

The playbook implements the following tasks:
- Launch a new Docker container
- Set max memory count
- Install Python3
- Install Docker Python3 container
- Install and launch Elk Server container

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

Project 1/Screen Shots/docker_ps.png

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
Web 1-3 at 10.0.0.7, 10.0.0.9 and 10.0.0.11 respectfully.

We have installed the following Beats on these machines:
- Metricbeat and Filebeat

These Beats allow us to collect the following information from each machine:
- Filebeat collects all system event logs to monitor for suspicious activity and monitoring of overall system health. 
- Metricbeat collects all statuses of system components and virtual hardware used to monitor CPU, RAM Memory and Network activity. 

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the -curl funtion to download the git files to the destination machine.
- Update the hosts file to include the private IP's of the destination machines.
- Run the playbook, and navigate to the Kibana webpage on the Elk server to check that the installation worked as expected.


- _Which file is the playbook? Where do you copy it? 
    - Playbooks are designated by a file extension of .yml, you copy it into your ansible container. 
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on? 
    - You update the hosts file to designate which machines the playbook will install software. In the hosts file you have to designate a [hosts] server section and a seperate [elk] server section with the appropriate IP's. Then in your playbook you would designate which host group you are pushing the data to. 
- _Which URL do you navigate to in order to check that the ELK server is running? 
    - http://<public_ip>:5601/apps/kibana

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
