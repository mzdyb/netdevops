# NetDevOps with Ansible Automation Platform

This project shows the example of how can we automate network configuration using Ansible Automation Platform and CI/CD approach. The core assumption here is that we are moving Source of True for our network configuration to GitHub. Devices configurations are defined by configuration variables in yaml files and by configuration logic scripted in jinja2 files. To generate configurations and apply them to devices Ansible network config module is used. I am using here intended configuration approach which means that whole device's configuration is generated before pushing it to device. Ansible config module (eos_config in this case) by using internal NOS diff mechanism applies only the config diff (so idempotency is ensured meaning no configuration overwriting happens).



## Network environment

In this case environment consists of two separate networks: Developement and Production. They have the same topologies and configurations. The same SoT for both Developement and Production networks is used to avoid the need for synchronization of two separate SoTs. But in case of Developement network there is adjustment of hostnames (for better visibility which device are we using) and Management interface's IP and static routes allowing connectivity to Developement network. Networks are implemented using Containerlab and consist of two Arista routers and two Linux clients
  
Production Network:  
![a01-prod](https://github.com/mzdyb/netdevops/assets/49950423/ca8ca593-66c2-4054-b994-69f7f22ff288)

Developement Network:  
![a01-dev](https://github.com/mzdyb/netdevops/assets/49950423/a114ab2b-c5a1-4c39-9d7d-7aa0296b50b5)



## Workflow
The core idea here is that GitHub is the Source of True for our network and any changes to the network are performed from Git. The Production network's configuration is represented by main branch. 

**CI pipeline**

Whenever we want to add configuration change in the network the new branch should be created. The new branch's name should start with 'cfg_updates' string. After staging and commiting changes we are pushing them to GitHub. When the name of branch starts with 'cfg_updates' push event triggers GitHub action 'CI' which in turn uses webhook to trigger CI Workflow template on Ansible Automation Platform.

    git checkout -b cfg_updates_bgp_updates
    git add topologies/arista01/a01-prod-rtr1/config_vars/bgp.yml
    git commit -m 'added prefix 172.16.0.1/32 to bgp'
    git push -u origin cfg_updates_bgp_updates

Below we can see CI Workflow created on Ansible Automation Platform:
![AAP_CI_workflow](https://github.com/mzdyb/netdevops/assets/49950423/ef7a2ac2-fce6-4b51-8dbb-1395dba8aa95)


It consists of three stages:
1. Synchronize SoT to have the latest repo version
2. Configure Developement network with configuration from 'cfg_updates_bgp_updates' branch
3. Testing
  
  
**CD pipeline**  

When the new configuration has been tested sucessfully network engineer may want to deploy these changes to Production network. In this case Pull Request should be created in GitHub:

1. Create Pull Request

![create_pull_request](https://github.com/mzdyb/netdevops/assets/49950423/e277cf52-8d6e-4f11-8c7b-ea1a53b6c48b)

 
2. Review configuration changes in Pull Request (in this example prefix 172.16.0.1/32 has been added)
   
![review_configuration_changes](https://github.com/mzdyb/netdevops/assets/49950423/ac79e94a-ef22-426e-900b-18f5942762bf)

   
3. Merge changes to main branch and close Pull Request

![merge_and_close_pr](https://github.com/mzdyb/netdevops/assets/49950423/262b7058-fb26-48d2-bbef-402d12e8914f)



When Pull Request is merged and closed GitHub action 'CD' is triggered which in turn uses webhook to trigger CD Workflow template on Ansible Automation Platform:
![AAP_CD_workflow](https://github.com/mzdyb/netdevops/assets/49950423/41349fe8-d247-484b-b81c-7ebcd5cb81bd)


As we can see this is the classic network deployment approach with pre-checks, configuration changes, post-checks and possible rollback but with the major difference: here everything is fully automated. The crucial part of implementing network configuration changes is creating backup and rollback plan. In this case backup of running configuration is created as a part of 'configure prod network' block so is not shown in the above workflow. But because we moved SoT for our network to Git we don't even have to use this backup because we have fully tracked and version controlled configuration on GitHub. So rollback in this case means reverting main branch to commit before merging changes from 'cfg_updates_bgp_updates' branch and applying configuration state from this commit.

**Notifications**  
Each template in both workflows has Notification functionality enabled on Ansible Automation Platform. It means that the target audience (for example a group of network engineers and service managers) can be informed about the status of every step in the workflow. I am using here Slack bots app, see below the example of automated messages sent from Ansible Automation Platform to #netdevops Slack channel informing about every step of CI and CD workflows:
![slack](https://github.com/mzdyb/netdevops/assets/49950423/5ecf8279-e43a-4f02-95e1-bb0ffbdbe323)



**Testing**  
It this example we are implementing simple configuration change which is adding prefix 172.16.0.1/32 to BGP process. There are two kinds of testing targets here: the network itself (two Arista container based routers) and two Linux container based clients for traffic simulation. In the network itself we are testing if the new prefix has been added to the routing table and if BGP session between routers is in Established state. With Linux clients we are veryfying connectivity using ping. We can easily verify automated rollback and simulate outage by for example changing BGP AS to incorrect value in main branch before running AAP CD workflow.
  

## Final Remarks
This project shows the example of one of the NetDevOps use cases which is moving SoT to GitHub and use automated testing and deployment of configuration changes. It should not be confused with full CI/CD workflow like we have in software developement realm. The major challenge in implementing full CI/CD in networking realm is lack of Developement environment which would 1:1 reflect Production environment. Physical lab is very expensive so it is almost never the case to have identical Developement environment like Production and virtual devices are not always available and in practice they don't have identical functionality like their hardware counterparts. Virtual devices are good for emulating control plane but data plane is implemented using ASICs and not all of its functions are virtualized. But Developement and Production workflows in this example are exactly the manual workflows which are often used by network engineers in their manual network changes preparation and implementation even if they have access to limited lab. By using the benefit of moving SoT to central place and Ansible automation workflows inspired by CI/CD approach we can automate these network activities and showing the example how it can be done is the purpose of this project.   

There is a tendency in the industry to call workflows with SoT moved to GitHub as 'GitOps' but one of the core principles of GitOps is that infrastructure/configuration should be continuously reconciled. This is not the case with workflows I am presenting here so I am not calling this approach GitOps. Continuous reconciliation might be implemented with Event-Driven Ansible and AAP Scheduler but this is the topic for another project.

I am using here intended configuration approach which requires to script whole configuration with jinja2. We can reduce the number of j2 files by rendering some sections of configuration with [Network Resource Modules](https://docs.ansible.com/ansible/latest/network/user_guide/network_resource_modules.html#network-resource-modules). I will add to the project this approach in the future.

## Feedback
Feedback is always welcome! If you have any comments, please reach me out

## Author

[@mzdyb](https://www.linkedin.com/in/michal-zdyb-9aa4046/)

