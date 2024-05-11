# NetDevOps with Ansible Automation Platform

This project shows the example of how can we automate network configuration using Ansible Automation Platform and CI/CD approach. The core assumption here is that we are moving Source of True for our network configuration to GitHub. Devices configurations are defined by configuration variables in yaml files and by configuration logic scripted in jinja2 files. To generate configurations and apply them to devices Ansible network config module is used. I am using here intended configuration approach which means that whole device's configuration is generated before pushing it to device. Ansible config module (eos_config in this case) by using internal NOS diff mechanism applies only the config diff (so idempotency is ensured meaning no configuration overwriting happens).



## Network environment

In this case environment consists of two separate networks: Developement and Production. They have the same topologies and configurations. The same SoT for both Developement and Production networks is used to avoid the need for synchronization of two separate SoTs. But in case of Developament network there is adjustment of hostnames (for better visibility which device are we using) and Management interface's IP and static routes allowing connectivity to Developement network. Networks are implemented using Containerlab and consist of two Arista routers and two Linux clients
  
Production Network:  
![a01-prod](https://github.com/mzdyb/netdevops/assets/49950423/ca8ca593-66c2-4054-b994-69f7f22ff288)

Developement Network:  
![a01-dev](https://github.com/mzdyb/netdevops/assets/49950423/a114ab2b-c5a1-4c39-9d7d-7aa0296b50b5)



## Workflow
The core idea here is that GitHub is the Source of True for our network and any changes to the network are performed from Git. The Production network configuration is represented by main branch. 

**CI pipeline**

Whenever we want to add configuration change int the network the new branch should be created. The new branch's name should start with 'cfg_updates' string. After staging and commiting changes we are pushing them to GitHub. When the name of branch starts with 'cfg_updates' push event triggers GitHub action 'CI' which in turn uses webhook to trigger CI Workflow template on Ansible Automation Platform.

    git checkout -b cfg_updates_bgp_updates
    git add topologies/arista01/a01-prod-rtr1/config_vars/bgp.yml
    git commit -m 'added prefix 192.168.100.0/24 to bgp'
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

![create_pull_request](https://github.com/mzdyb/netdevops/assets/49950423/8caab6c5-a408-4c65-b156-27169195a057)
 
2. Review configuration changes in Pull Request (in this example prefix 192.168.100.0/24 has been added)
   
![review_configuration_changes](https://github.com/mzdyb/netdevops/assets/49950423/9dcc77e0-64a8-42db-8f49-7a148f9f802a)

   
3. Merge changes to main branch and close Pull Request

![merge_and_close_pr](https://github.com/mzdyb/netdevops/assets/49950423/a3865106-21b9-45e1-865b-20d5d2f8a1b8)

When Pull Request is merged and closed GitHub action 'CD' is triggered which in turn uses webhook to trigger CD Workflow template on Ansible Automation Platform:
![AAP_CD_workflow](https://github.com/mzdyb/netdevops/assets/49950423/58882b63-c026-43f9-9d74-70879ea556a0)

As we can see this is the classic network deployment approach with pre-checks, configuration changes, post-checks and possible rollback but with the major difference: here everything is fully automated. The crucial part of implementing network configuration changes is creating backup and rollback plan. In this case backup of running configuration is created as a part of 'configure prod network' template so is not shown in the above workflow. But because we moved SoT for our network to Git we don't even have to use this backup because we have fully tracked and version controlled configuration on GitHub. So rollback in this case means reverting main branch to commit before merging changes from 'cfg_updates_bgp_updates' branch and applying configuration state from this commit.

**Notifications**  
Each template in both workflows has Notification functionality enabled on Ansible Automation Platform. It means that the target audience (for example a group of network engineers and service managers) can be informed about the status of every step in the workflow. I am using here Slack bots app, see below the example of automated messages sent from Ansible Automation Platform to #netdevops Slack channel informing about every step of CI and CD workflows:
![slack](https://github.com/mzdyb/netdevops/assets/49950423/be27c5cb-c179-45a1-9fc6-b9c5af04ac68)


**Testing**  
It this example we are implementing simple configuration change which is adding local prefix 192.168.100.0/24 to BGP process. It doesn't bring any new functionality to the network just shows the process of introducing new configuration to devices during CI and CD phases. There are two kinds of testing targets here: the network itself (two Arista container based routers) and two Linux container based clients for traffic simulation. In the network itself we are testing if BGP session between routers is in Established state, in Linux clients we are veryfying connectivity with ping. We can easily verify automated rollback and simulate outage by for example changing BGP AS to incorrect value in main branch before running AAP CD workflow.
  
...  
...  
...

## Final Remarks



## Author

[@mzdyb](https://www.linkedin.com/in/michal-zdyb-9aa4046/)

