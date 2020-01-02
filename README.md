
# Infrastructure Automation using Ansible v1.0
This repository is created to automate the repetitive daily tasks for Infrastructure Engineers on multiple platforms.
In v1.0, the automation is focusing mainly on ACI automation, and work is currently in progress to include more playbooks, and roles for automating tasks on NXOS, UCS, ...etc.


As of v1.0, the supported playbooks are:

- pb-aci-access-policies.yml                       >>> Create/delete all Access Policies on ACI (VLAN Pool, VLP Encap Block, Domain, VLP to DOM association, AEP, Intf Policies, IPG, IPR, IST, SPF, SST, and vPC Policies )
- pb-aci-attachable-entity-profile.yml               >>> Create/delete AEPs only
- pb-aci-bd-and-EPG.yml                              >>> Create/delete BD and EPGs
- pb-aci-bd-assign-subnet.yml                        >>> Create/delete BD Subnet
- pb-aci-domain-and-vlan.yml                         >>> Create/delete (VLAN Pool, VLP Encap Block,and Domain)
- pb-aci-epg-query-all.yml                           >>> Query all EPGs in all tenants
- pb-aci-epg-static-binding.yml                      >>> Create/delete EPG Static Binding (Access, PC, or vPC)
- pb-aci-epg-to-domain.yml                           >>> Create/delete Domain to EPG Association/binding
- pb-aci-interface-profile.yml                       >>> Create/delete interface profiles
- pb-aci-physical-domain-to-vlan-binding.yml           >>> Create/delete Domain to EPG Association/binding
- pb-aci-physical-domain.yml                           >>> Create/delete Physical Domains
- pb-aci-policy-group.yml                              >>> Create/delete Policy Groups
- pb-aci-switch-profile.yml                            >>> Create/delete Switch Profiles
- pb-aci-tenant-vrf-ap.yml                             >>> Create/delete Tenant, VRF, and APs
- pb-aci-tenant-vrf.yml                                >>> Create/delete Tenant, and VRFs
- pb-aci-tenant-vrf-bd-bd_subnet.yml                   >>> Create/delete Tenant, VRF, BD, and BD Subnets  
- pb-aci-vlan-pool.yml                                 >>> Create/delete VLAN Pools
- pb-aci-vpc-protection-group.yml                      >>> Create vPC Protection Group Policies


In the same repo, there are 28 roles that can be used to create more playbooks/use cases. (Please refer to "Create your own Playbook" section)




# Content Structure/Organization

Directory layout

├── ACI_HOSTS                   # Inventory file storing all the targets (INI format)
├── README.md
├── ansible.cfg                 # Ansible Configuration file used for storing related parameters to this repo
│
├── environments                # System specific variables
│   └── ACI
│     └── Customer-01           # Variables created specifically for Customer-01
│       ├── Test-TN.yml         # Customer-01, Test-TN   (Contains all the config parameters "logical constructs", Tenant Name, EPGs' names, Contracts... etc)
│       └── fabric.yml          # Customer-01, Test-TN   (Contains all the config parameters "Access/fabric Policies", VLAN Pools, Domains, AEPs, ....etc)
│
├── group_vars                  # Here we assign variables to a particular group of targets (for ex.communication ports, )
│   ├── aci-CUST-01             # Variables created specifically for Customer-01
│   └── aci-CUST-02             # Variables created specifically for Customer-02
│
├── plays                       # Playbooks Library
│   ├──  pb-aci-XX.yml          # All the playbooks are creating or deleting objects/configs. XX refers the actions will be executed on the targets.
│   └──  pb-aci-XX-query.yml    # All the playbooks that have "-query" are for querying specific objects. (for ex. Querying all the VLAN Pools..etc)
│
├── plugins                     
│   └── filter
│     └──  aci.py               # Custom jinja filter.
│
└── roles                       # Roles Library
    └──  aci-XX                 # Each role execute a specific task, and has a "tasks" directory.
      └──  tasks        
        └──  main.yml           # Each "main.yml" file  recalls Ansible Modules. Ansible Modules has a naming format "aci_xx"

# Notes

- All the playbooks have the below characteristics:  
  - Represents a use case (for ex. Playbook deploys All ACI Interface Policies, or All ACI Access policies).
  - Recalls the targets/hosts "which is defined through the Ansible command as extra_vars". (for ex. target=ACI_HOSTS)
  - Recalls the variables "configurations parameters" stored in ./environment.
  - Recalls one or more roles.

- All targets (hosts) are stored in Inventory file named [ACI_HOSTS]. all similar hosts can be grouped and named for easier management of the hosts, these groups are created are recalled in [./group_vars] folder, and any modification to the group name in ACI_HOSTS file has to be matched in the [./group_vars] folder.

- All the roles have the below characteristics:
  - Most of the roles are based on new Ansible modules v2.6
  - All roles recall the target_IP, target_username, and target_password which stored in "./group_vars/aci-CUST-XX"

- Refer to the Tenant Configs Template "./environment/ACI/Customer-01/Test-TN", and Fabric/Access Policies Configs Template "./environment/ACI/Customer-01/fabric.yml"

# Starting Ansible?  
- Introductory trainings
   https://www.ansible.com/webinars-training/introduction-to-ansible
   https://www.ansible.com/webinars-training/automating-your-network
   https://www.udemy.com/ansible-for-network-engineers-cisco-quick-start-gns3-ansible/
- Advanced Trainings
   https://www.udemy.com/learn-ansible-advanced/
   https://www.udemy.com/mastering-ansible-x/

- Play with Ansible (installation guide - https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)


# How to start?

- Start with running the playbooks as below:

For Deploying configs
ansible-playbook -i ACI_HOSTS ./plays/pb-aci-XX.yml --extra-vars "ACI_Fabric=../environments/ACI tenant_name=Test-TN target=aci-CUST-01" -c local

For Removing configs
ansible-playbook -i ACI_HOSTS ./plays/pb-aci-XX.yml --extra-vars "ACI_Fabric=../environments/ACI tenant_name=Test-TN target=aci-CUST-01 action=delete" -c local

For Querying
ansible-playbook -i ACI_HOSTS ./plays/pb-aci-XX-query.yml --extra-vars "ACI_Fabric=../environments/ACI tenant_name=Test-TN target=aci-CUST-01" -c local


# Create your own Playbook

- Since playbooks are representing a specific use cases that will defer based on each customer design. Accordingly, the initial playbooks included in v1.0 are the ones used in the first deployment that may not cover all the other use cases for other customers.

- Playbooks can have numerous combinations of tasks "roles" to be executed on ACI.

- Creating a playbook is an easy task

For Playbooks related to Fabric/Access policies

```
- name: On Boarding ACI Access Policies  <<< Use proper naming for your playbook to easily identify the objective of this pb
  hosts: "{{ target }}"        <<< recalls the target hosts from the inventory  "Mandatory field"
  gather_facts: no


  tasks:
    - include_vars:
        file: "{{ ACI_Env_Path }}/fabric.yml"  <<< All the variables related to fabric/access policies
        name: ACI_Fabric

    - include_role:
        name: aci-vlan-pool  <<< refers to the name of the role "folder name", replace/modify the name of the role based on the task you want to execute on the targets.
```

For Playbooks related to Tenant configurations

```
- name: On Boarding ACI Access Policies  <<< Use proper naming for your playbook to
  hosts: "{{ target }}"
  gather_facts: no

  tasks:
    - include_vars:
        file: "{{ ACI_Env_Path }}/{{ tenant_name }}.yml"    <<< All the variables related to Tenant configs
        name: ACI_Fabric

    - include_role:
        name: aci-tenant   <<< refers to the name of the role "folder name", replace/modify the name of the role based on the task you want to execute on the targets.
```

- Notice that the difference will be only in which variables file to be called out. (fabric.yml OR Customer-01.yml)
