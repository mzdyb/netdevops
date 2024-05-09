# NetDevOps with Ansible Automation Platform

This project shows the example how can we automate network configuration using Ansible Automation Platform and CI/CD approach. The core assumption here is that we are moving Source of True for our network configuration to GitHub. Devices configurations are defined by configuration variables in yaml files and by configuration logic scripted in jinja2 files. To generate configurations and apply them to devices Ansible network config module is used.



## Network environment

Environment consists of two separate networks: Developement and Production. They have the same infrastructure and configurations
## Workflow
...
