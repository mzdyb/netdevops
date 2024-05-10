# NetDevOps with Ansible Automation Platform

This project shows the example how can we automate network configuration using Ansible Automation Platform and CI/CD approach. The core assumption here is that we are moving Source of True for our network configuration to GitHub. Devices configurations are defined by configuration variables in yaml files and by configuration logic scripted in jinja2 files. To generate configurations and apply them to devices Ansible network config module is used.



## Network environment

Environment consists of two separate networks: Developement and Production. They have the same topologies and configurations. The same SoT for both Developament and Production networks is used to avoid the need of two separate SoTs synchronization. But in case of Developament network there is adjustment of hostnames (for better visibility which device are we using) and Management interface's IP and static routes allowing connectivity to Developement network. Networks are implemented using Containerlab and consist of two Arista routers and two Linux clients
  
Developement Network:  
![a01-dev](https://github.com/mzdyb/netdevops/assets/49950423/ef10f61e-c031-4e5c-af29-f3d276da5be7)

Production Network:  
![a01-prod](https://github.com/mzdyb/netdevops/assets/49950423/71b64fab-5671-4e49-829b-a05121719f67)




## Workflow
...
