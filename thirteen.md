# Ansible Dynamic Assignments (Include) and Community Roles

In this project(which is a continuation of Project 12) we will be introduced to dynamic assignments by using include module.
What is the difference between static and dynamic assignments?

Static assignments use import Ansible module while Dynamic assignments use include Ansible module.

Hence,

import = Static
include = Dynamic

When the import module is used, all statements are pre-processed at the time playbooks are parsed. This means that when you execute site.yml playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when include module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the
statements encountered during execution will be used.

Take note that in most cases it is recommended to use static assignments for playbooks, because it is more reliable. With dynamic ones, it is hard to debug playbook problems due
to its dynamic nature. However, you can use dynamic assignments for environment specific variables as we will be introducing in this project.

In the  GitHub repository we have been usong since Project 11, start a new branch and give it a new name( named mine *thirteen* ).

Create a new folder, name it dynamic-assignments. Then inside this folder, create a new file and name it *env-vars.yml*. We will instruct *site.yml* to include this playbook later. For now, let us keep building up the structure.

Your GitHub should have the following structure:

├── dynamic-assignments

    └── env-vars.yml

├── inventory

    └── dev
    
    └── stage
    
    └── uat
    
    └── prod
    
└── playbooks

    └── site.yml
└── roles (optional folder)

    └──...(optional subfolders & files)
    
└── static-assignments

    └── common.yml
    
    
Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.

For this reason, we will now create a folder to keep each environment’s variables file. Therefore, create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables.

Your layout should now look like this.

├── dynamic-assignments

    └── env-vars.yml
   
├── env-vars

    └── dev.yml
    
    └── stage.yml
    
    └── uat.yml
    
    └── prod.yml
    
├── inventory

    └── dev
    
    └── stage
    
    └── uat
    
    └── prod
    
├── playbooks

    └── site.yml
    
└── static-assignments

    └── common.yml
    
    └── webservers.yml

Now paste the instruction below into the env-vars.yml file.

---
- name: collate variables from env specific file, if it exists

  hosts: all

  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - uat.yml
            - prod.yml
            - stage.yml
            - dev.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always


Notice 3 things to notice here:

We used include_vars syntax instead of include, this is because Ansible developers decided to separate different 

features of the module. From Ansible version 2.8, the include module is deprecated and variants of *include_* must be 

used. These are:

- include_role

- include_tasks

- include_vars


In the same version, variants of import were also introduced, such as:

- import_role

- import_tasks


We made use of a special variables *{{ playbook_dir }}* and *{{ inventory_file }}*. *{{ playbook_dir }}* will help 

Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem.

*{{ inventory_file }}* on the other hand will dynamically resolve to the name of the inventory file being used, then

append *.yml* so that it picks up the required file within the *env-vars* folder.



We are including the variables using a loop. *with_first_found* implies that, looping through the list of files, the

first one found is used. This is good so that we can always set default values in case an environment specific env 

file does not exist.

Update *site.yml* with dynamic assignments

Update *site.yml* file to make use of the dynamic assignment. (At this point, we cannot test it yet. We are just 

setting the stage for what is yet to come. So hang on to your hats)

*site.yml* should now look like this.

---
- name: Include dynamic variables 

  hosts: all

  tasks:
    - import_playbook: ../static-assignments/common.yml 
    - include_playbook: ../dynamic-assignments/env-vars.yml

  tags:
    - always

- name: Webserver assignment

  hosts: webservers

  import_playbook: ../static-assignments/webservers.yml

## Community Roles

Now it is time to create a role for MySQL database - it should install the MySQL package, create a database and 

configure users. There are tons of roles that have already been developed by other open source engineers out there.

These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy, we 

can simply download a ready to use ansible role, and keep going.

Download Mysql Ansible Role

You can browse available community roles at *galaxy.ansible.com*

We will be using a MySQL role developed by *geerlingguy*.