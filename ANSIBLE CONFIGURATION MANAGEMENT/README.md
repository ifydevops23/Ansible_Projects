# ANSIBLE CLIENT AS A JUMP SERVER (BASTION HOST)
A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided.<br> 
If you think about the current architecture you are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provide better security and reduces attack surface.<br>
On the diagram below the Virtual Private Network (VPC) is divided into two subnets – Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses. <br>
For now, we will develop Ansible scripts to simulate the use of a Jump box/Bastion host to access our Web Servers.<br>

# Task 
- Install and configure Ansible client to act as a Jump Server/Bastion Host.(control Server for Managed Hosts(WebServers,DB,NFS Server))
- Create a simple Ansible playbook to automate servers configuration.

## STEP 1 - INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE <br>
** Prerequisite: Have Jenkins Installed in an EC2 server with port 8080 open.**
- Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.
A. Install Ansible <br>
`sudo apt update`<br>
`sudo apt install ansible`<br>
![1_install_ansible](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/252ea927-8f9f-4af8-bac4-3120d33c6cfc)
- Check your Ansible version by running ansible --version
![1_ansible_version](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/1622befd-1314-4c32-9518-c97b911f7874)


B. In your GitHub account create a new repository and name it ansible-config-mgt.

![1_create_repository](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/31c4975c-ba6a-40a1-9c8c-f213f6078c62)
**Configure Jenkins build job to save your repository content every time you change it.**
- Create a new Freestyle project 'ansible' in Jenkins and point it to your ‘ansible-config-mgt’ repository.
- ![1_create_freestyle_project](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/58fda944-1a5f-4e59-914d-95f4b0bc0b05)
- Configure Webhook in GitHub.
![1_webhook_created](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/4d617009-27ce-4508-909c-13372d4af3b7)
- Set webhook to trigger 'ansible' build.
![2_add_repository_link](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/ea53e87d-20ee-463b-bfbf-314a300cd9b2)
![2_add_git_build_triggers](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/49ad87e5-4bce-4cf4-b784-9d43b56940d7)
- Configure a Post-build job to save all (**) files.
![2_post_build_action](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/ee4d1e0f-2882-45d9-9d6d-1826c6f379bf)
- Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder<br>
```
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```
**Note: Trigger Jenkins project execution only for /main (master) branch**.<br>
**Note: Allocate an Elastic IP to your Jenkins-Ansible server.**

STEP 2 – PREPARE YOUR DEVELOPMENT ENVIRONMENT USING VISUAL STUDIO CODE <br>
First part of ‘DevOps’ is ‘Dev’, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable.

- After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.<br>

## STEP 3 - BEGIN ANSIBLE DEVELOPMENT<br>
- In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature. <br>
Tip: Give your branches descriptive and comprehensive names, for example, if you use Jira or Trello as a project management tool – include ticket number (e.g. PRJ-145) in the name of your branch and add a topic and a brief description what this branch is about – a bugfix, hotfix, feature, release (e.g. feature/prj-145-lvm) <br>
- Checkout the newly created feature branch to your local machine and start building your code and directory structure `git checkout -b <branchname>`
![3_github_create_newbranch_from_vscode](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/e9ce85a3-e9e4-4886-9243-a6322700f913)
- Create a directory and name it playbooks – it will be used to store all your playbook files.
-- Within the playbooks folder, create your first playbook, and name it common.yml
- Create a directory and name it inventory – it will be used to keep your hosts organised.
- Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.
![3_files_uat_dev_prod_staging_created](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/ed504e46-4dbb-4ad0-84a4-7fc2d3ecad51)

## Step 4 – SET UP AN ANSIBLE INVENTORY<br>
An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate.<br>
Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.<br>

- Setup SSH agent and connect VS Code to your Jenkins-Ansible instance: <br>
```
eval `ssh-agent -s`
```
`ssh-add <path-to-private-key>` <br>
Confirm the key has been added with the command below, you should see the name of your key <br>
`ssh-add -l`<br>
Now, ssh into your Jenkins-Ansible server using ssh-agent <br>
`ssh -A ubuntu@public-ip`<br>
Confirm again the key has been added with the command below, after you ssh: <br>
`ssh-add -l`<br>

- Save below inventory structure in the inventory/dev file to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.

Update your inventory/dev.yml file with this snippet of code:

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```
**Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent:**


## STEP 5 – CREATE A COMMON PLAYBOOK <br>
It is time to start giving Ansible the instructions on what you need to be performed on all servers listed in inventory/dev.<br>
In common.yml playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.<br>
Update your playbooks/common.yml file with following code:<br>

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark-qt
        state: latest
```

![7_playbook_common_yml](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/d4a99f31-ab05-46c8-8e0e-14fe261eff4b)

This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers.<br>
It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.<br>

## STEP 6 – Update GIT with the latest code <br>
Now all of your directories and files live on your machine and you need to push changes made locally to GitHub.
Now you have a separate branch, you will need to know how to raise a Pull Request (PR), get your branch peer reviewed and merged to the master branch.

- Commit your code into GitHub:
- use git commands to add, commit and push your branch to GitHub.
`git status`<br>
`git add <selected files>`<br>
`git commit -m "commit message"`<br>

- Create a Pull request (PR)
![8_git_push_succesful](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/bf4ba183-a73d-456f-b028-323d6cc8e4e7)

![8_created_a pull_request](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/98c315c0-4751-41f6-8a98-27e596a52b8d)

![8_merge_pull_request](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/964691bf-046d-4cff-b2a2-087336271d32)

- Wear a hat of another developer for a second, and act as a reviewer.

- If the reviewer is happy with your new feature development, merge the code to the master branch.
- Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.<br>
![8_git_pull](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/7c025c20-8601-40d2-a556-d77e662746e6)
Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server.<br>

![8_checked_artifacts](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/1a28710f-1878-48c9-adf3-66e21e07fa93)

Step 7 - RUN FIRST ANSIBLE TEST <br>
Now, it is time to execute ansible-playbook command and verify if your playbook actually works: <br>
- Clone down your ansible-config-mgt repo (latest) to your Jenkins-Ansible instance<br>
`git clone <ansible-config-mgt repo link>`<br>

![3_clone_down_ansible_config_repo_to_instance](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/31967bb8-a890-442f-bfa6-afc499f6891b)

`cd ansible-config-mgt`<br>
`ansible-playbook -i inventory/dev.yml playbooks/common.yml`<br>

ALTERNATIVELY; <br>
```
cd /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
ansible-playbook -i inventory/dev.yml playbooks/common.yml

```
![5_succesful_play_run](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/78648bc3-7c45-45b1-9172-515f46d3d8ee)

- You can go to each of the servers and check if wireshark has been installed by running `which wireshark` or `wireshark --version`<br>
**SERVER 1 - remote_user=ec2-user<br>

![5_which_wireshark_ec2-user](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/6d8fd3f5-2f9a-42ee-9c31-513d93604a45)

**SERVER 2 - remote_user=ubuntu<br>

![5_which_wireshark_ubuntu_user](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/8a54a2cb-7c29-48aa-a551-d71eda2234c5)

**Optional step – Repeat once again**<br>

- Update your ansible playbook with some new Ansible tasks
- Create a directory and a file inside it.
```
  - name: create directory
    file:
      path: /home/ubuntu/storage
      state: directory
  - name: create file
    file:
      path: /home/ubuntu/storage/file.txt
      state: touch
```

- and go through the full checkout -> change codes -> commit -> PR -> merge -> build -> ansible-playbook cycle again to see how easily you can manage a servers fleet of any size with just one command!

![6_optional_create_file_and_folder](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/a60f4d5d-8e12-4dfd-9e29-e54e81da310d)

- Confirm changes
**SERVER-Load Balancer <br>

![11_file_and_folder_created](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/bedb8f3e-d2ba-4e01-9efb-c04946b4462e)

**SERVER-Web-server <br>

![11_file_and_folder_created_ec2-user](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/d85899f0-3ea6-42bd-a0c5-a082eb1baae7)




