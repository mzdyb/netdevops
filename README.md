ork# NetDevOps with Ansible Automation Platform

This project shows the example how can we automate network configuration using Ansible Automation Platform and CI/CD approach. The core assumption here is that we are moving Source of True for our network configuration to GitHub. Devices configurations are defined by configuration variables in yaml files and by configuration logic scripted in jinja2 files. To generate configurations and apply them to devices Ansible network config module is used.



## Network environment

Our environment consists of two separate networks: Developement and Production. They have the same topologies and configurations. The same SoT for both Developament and Production networks is used to avoid the need for two separate SoTs synchronization. But in case of Developament network there is adjustment of hostnames (for better visibility which device are we using) and Management interface's IP and static routes allowing connectivity to Developement network. Networks are implemented using Containerlab and consist of two Arista routers and two Linux clients
  
Production Network:  
![a01-prod](https://github.com/mzdyb/netdevops/assets/49950423/ca8ca593-66c2-4054-b994-69f7f22ff288)

Developement Network:  
![a01-dev](https://github.com/mzdyb/netdevops/assets/49950423/a114ab2b-c5a1-4c39-9d7d-7aa0296b50b5)



## Workflow
The core idea here is that GitHub is the Source of True for our network and any changes to the network are performed from Git. The Production network configuration is represented by main branch. 

**CI pipeline**

Whenever we want to add configuration change int the network the new branch should be created. The new branch's name should start with 'cfg_updates' string. After staging and commiting changes we are pushing them to GitHub. When the name of branch starts with 'cfg_updates' push event triggers GitHub action CI which in turn uses webhook to trigger CI Workflow template on Ansible Automation Platform.

    git checkout -b cfg_updates_bgp_updates
    git commit - m 'added new prefix'
    git push -u origin cfg_updates_bgp_updates

Below we can see CI Workflow created on Ansible Automation Platform:
![AAP_CI_workflow](https://github.com/mzdyb/netdevops/assets/49950423/777cf07c-6f78-424a-94c0-b065bae1e76d)

It consists of three stages:
1. Synchronize SoT to have the latest repo version
2. Configure Developement network with configuration from 'cfg_updates_bgp_updates' branch
3. Testing

...  
...  

## Author

[@mzdyb](https://www.linkedin.com/in/michal-zdyb-9aa4046/)

