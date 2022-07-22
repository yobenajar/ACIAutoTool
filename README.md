# ACIAutoTool

## Introduction
Tool for automating configuration tasks in ACI. Eliminates manual use of APIC for configuration. 
You can configure multiple Interfaces, EPGs, Static Paths Bindings and a host of other things by simply adding data to a table.
NB!! The setup of this tool is quite heavy, but easy to use once set up.

Technologies used: 'Ansible', 'Python', ''

## Setup
###  Ansible


##### Ansible File Structure
```
<span style="color:red">some **aci-fabric** text</span>
└───playbook and data
│    | inventory
|    | aci_playbook.yml
|    | aci_data.yml
|
└───roles
│   └─── aci_tenant_construct_layer2
│         └─── tasks
│         | aci_epg.yml
|         | aci_bd.yml
|         | aci_vrf.yml
│         | ...
```
