# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

In this project you will continue working with ansible-config-mgt repository and make some improvements of your code. Now you need to refactor your Ansible code, create assignments, and learn how to use the imports functionality. Imports allow to effectively re-use previously created playbooks in a new playbook – it allows you to organize your tasks and reuse them when needed.<br>

**Code Refactoring**<br>
Refactoring is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. 
The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.
In your case, you will move things around a little bit in the code, but the overal state of the infrastructure remains the same.<br>
Let us see how we can improve your Ansible code!

## Step 1 – Jenkins job enhancement
Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place.
Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job – we will require Copy Artifact plugin.<br>
Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store there all artifacts after each build.<br>

`sudo mkdir /home/ubuntu/ansible-config-artifact`<br>
![1_mkdir_cop-artifacts](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/dae67fe1-f507-4cf5-9ed9-78c3b9c7048b)

Change permissions to this directory, so Jenkins could save files there – <br>
`chmod -R 0777 /home/ubuntu/ansible-config-artifact`<br>
![1_change_permission](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/cdf3af59-8b0a-49b6-9395-0fec67268672)

Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins.<br>
![1_install_without_restart_copy_artifacts](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/bb0eedab-0b11-4e50-bbeb-4efda03e27b9)

Create a new Freestyle project and name it save_artifacts.<br>

![2_create_new_project_save_artifacts](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/25eb0f9d-6625-4deb-9dc4-21c5f044cb5c)

This project will be triggered by completion of your existing ansible project. Configure it accordingly:<br>

![2_configure_new_project_Build_triggers](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/65a07148-1bc7-41af-9323-09c8b87f22e3)

Note: You can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your ansible job.

![2_configure_new_project_discard_old_builds](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/6d3835a9-6b25-4870-8d6e-b0cfdb70eb0c)


The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory. <br>
To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.<br>

![2_add_build_step_1](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/96de54a8-e2a6-44f7-8d12-415ea38d4138)


![2_add_build_step_2](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/a7a230c2-8cfa-4e6a-987c-8812cad07647)

Test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside master branch).<br>
If both Jenkins jobs have completed one after another – you shall see your files inside /home/ubuntu/ansible-config-artifact directory and it will be updated with every commit to your master branch.<br>

![3_copied_artifacte_to_save_artifacts](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/d54649ed-ce0f-4219-af31-505e545209b8)

Now your Jenkins pipeline is more neat and clean.<br>

## REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO SITE.YML <br>

## Step 2 – Refactor Ansible code by importing other playbooks into site.yml <br>

Before starting to refactor the codes, ensure that you have pulled down the latest code from master (main) branch, and created a new branch, name it refactor.<br>
DevOps philosophy implies constant iterative improvement for better efficiency – refactoring is one of the techniques that can be used, but you always have an answer to question "why?". Why do we need to change something if it works well?
In Project Ansicble configuration Managment, you wrote all tasks in a single playbook common.yml, now it is pretty simple set of instructions for only 2 types of OS, but imagine you have many more tasks and you need to apply this playbook to other servers with different requirements.<br>
In this case, you will have to read through the whole playbook to check if all tasks written there are applicable and is there anything that you need to add for certain server/OS families. Very fast it will become a tedious exercise and your playbook will become messy with many commented parts. Your DevOps colleagues will not appreciate such organization of your codes and it will be difficult for them to use your playbook.
Most Ansible users learn the one-file approach first.<br>
However, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.
Let see code re-use in action by importing other playbooks.<br>

Within playbooks folder, create a new file and name it site.yml <br>
This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. <br>
In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that you created previously. Dont worry, you will understand more what this means shortly.
Create a new folder in root of the repository and name it static-assignments. <br>
The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. You will see why the folder name has a prefix of static very soon. For now, just follow along.
Move common.yml file into the newly created static-assignments folder.<br>

![4_file_structure](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/01676687-3b2a-4357-a0fa-50855eef1731)

Your folder structure should look like this;<br>
```
├── static-assignments
│   └── common.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
```
- Inside site.yml file, import common.yml playbook.<br>
```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```
The code above uses built in import_playbook Ansible module.<br>

**Run ansible-playbook command against the dev environment**<br>
Since you need to apply some tasks to your dev servers and wireshark is already installed – you can go ahead and create another playbook under static-assignments and name it common-del.yml. In this playbook, configure deletion of wireshark utility.

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```

![5_create_and_edit_commom-del-yml](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/dc6a554e-dc74-4ac6-9c10-27ce746603ce)


- Update site.yml with - import_playbook: ../static-assignments/common-del.yml instead of common.yml and run it against dev servers:<br>

![5_2nd_edit_site-yml](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/cb3a14f5-905a-4f4d-8040-3eb3eeaba95b)

`cd /home/ubuntu/ansible-config-mgt/`

![6_cd_ansible_config_artifacts](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/b1a44b62-5cbe-4341-83a9-ea66a95adf1b)

`ansible-playbook -i inventory/dev.yml playbooks/site.yml`

![6_delete_wiresharK_usind_palybook](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/527d51fa-2fed-4022-ae02-e6168055685c)

Make sure that wireshark is deleted on all the servers by running wireshark --version <br>
Now you have learned how to use import_playbooks module and you have a ready solution to install/delete packages on multiple servers with just one command.

![6_confirm_delete](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/085de985-274f-4ed0-9e8a-589fa3281614)


## CONFIGURE UAT WEBSERVERS WITH A ROLE ‘WEBSERVER’<br>

## Step 3 – Configure UAT Webservers with a role ‘Webserver’ <br>
We have our nice and clean dev environment, so let us put it aside and configure 2 new Web Servers as uat.<br>
We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.<br>
**Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT.**<br>
Tip: Do not forget to stop EC2 instances that you are not using at the moment to avoid paying extra.<br>
For now, you only need 2 new RHEL 8 servers as Web Servers and 1 existing Jenkins-Ansible server up and running.<br>
To create a role, you must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory.<br>
There are two ways how you can create this folder structure:<br>
Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront)mkdir roles
```
cd roles
ansible-galaxy init webserver
```
Create the directory/files structure manually<br>
Note: You can choose either way, but since you store all your codes in GitHub, it is recommended to create folders and files there rather than locally on Jenkins-Ansible server.<br>

The entire folder structure should look like below, but if you create it manually – you can skip creating tests, files, and vars or remove them if you used ansible-galaxy
```
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml
```
After removing unnecessary directories and files, the roles structure should look like this<br>
```
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates
```

![0_file_structure](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/05dd293e-1352-4fcb-8593-330d09a15763)

**NOTE: Ensure you are using ssh-agent to ssh into the Jenkins-Ansible instance just as you have done in project Anble Configuration Management<br>

```
eval `ssh-agent -s`
ssh-add <name of key pair>
ssh -A username@Public-IP-Control-Node
ssh-add -l
```

![0_ssh_setup](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/3c880ace-219e-4e6c-978f-53640d6add7e)

**Copy your private key into a file in a known location**
Note: Locate location of downloaded key pair for example Downloads direcctory, then `cd Downloads` <br>

`cat <name-of-key-attached-to-managed-nodes>`

![0_private_key_pair_gen](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/1de2b4be-dfdf-4262-a69d-9b847d4bf530)

`ssh -A username@Public-IP-Control-Node`<br>

`sudo vi <desired-name>.pem`<br>

Paste key, save and note location `pwd`<br>

Update your inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of your 2 UAT Web servers<br>
```
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=/home/control-node-username/<name-of-key>
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=/home/control-node-username/<name-of-key>
```

![0_edit_uat_file](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/243492af-54ca-49fc-816b-db22aa6db61c)

In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory <br>
`roles_path = /home/ubuntu/ansible-config-mgt/static-assignments/roles`, so Ansible could know where to find configured roles.

![8_uncomment_roles_path](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/63c3336f-2ede-4a23-8df3-f347a64d45c3)

It is time to start adding some logic to the webserver role.
Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:<br>
- Install and configure Apache (httpd service)
- Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
- Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
- Make sure httpd service is started
Your main.yml may consist of following tasks:<br>
```
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```

![8_edit_task_main_yml](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/5e7e00b2-bf42-4e4b-8c3e-cccdf95af926)


## REFERENCE WEBSERVER ROLE <br>
## Step 4 – Reference ‘Webserver’  <br>
Within the static-assignments folder, create a new assignment for uat-webservers uat-webservers.yml. This is where you will reference the role.

```
---
- hosts: uat-webservers
  roles:
     - webserver
```

![9_create_uat_role](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/defcadd1-4409-4140-a963-73463b8b8c97)

Remember that the entry point to our ansible configuration is the site.yml file. Therefore, you need to refer your uat-webservers.yml role inside site.yml.
So, we should have this in site.yml<br>

```
---
- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml

```

## Step 5 – Commit & Test <br>
Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.

Now run the playbook against your uat inventory and see what happens:<br>
`sudo ansible-playbook -i /home/ubuntu/ansible-config-artifact/inventory/uat.yml /home/ubuntu/ansible-config-artifact/playbooks/site.yml`<br>

![9_success_from_ansible_playbook](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/501f0b75-8270-4839-91e3-dbfc302d9736)

You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:<br>
`http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php`<br>

![9_success_from_web-server_2](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/ad0127c1-b1b8-4e1f-8fa5-2d8c4511066c)

or
`http://<Web2-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php`<br>

![9_success_from_website](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/01f3149c-b279-49f4-9e02-7a435bc18331)

Your Ansible architecture now looks like this:<br>

![new_architecture](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/6a3f7069-7f41-4a28-90be-5b757936bb97)

Congratulations!<br>
You have learned how to deploy and configure UAT Web Servers using Ansible imports and roles!
