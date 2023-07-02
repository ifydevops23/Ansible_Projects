Ansible Client as a Jump Server (Bastion Host)

A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided.<br> 
If you think about the current architecture you are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provide better security and reduces attack surface.<br>
On the diagram below the Virtual Private Network (VPC) is divided into two subnets – Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses. <br>
For now, we will develop Ansible scripts to simulate the use of a Jump box/Bastion host to access our Web Servers.<br>

# Task 
- Install and configure Ansible client to act as a Jump Server/Bastion Host.(Master for slaves(WebServers,DB,NFS Server))
- Create a simple Ansible playbook to automate servers configuration.

## STEP 1 - INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE <br>
** Prerequisite: Have Jenkins Installed in an EC2 server with port 8080 open.**
- Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.
A. Install Ansible <br>
`sudo apt update`<br>
`sudo apt install ansible`<br>
- Check your Ansible version by running ansible --version

B. In your GitHub account create a new repository and name it ansible-config-mgt.
**Configure Jenkins build job to save your repository content every time you change it.**
- Create a new Freestyle project 'ansible' in Jenkins and point it to your ‘ansible-config-mgt’ repository.
- Configure Webhook in GitHub and set webhook to trigger 'ansible' build.
- Configure a Post-build job to save all (**) files.
- Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder<br>
```
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```
** Note: Trigger Jenkins project execution only for /main (master) branch.<br>
** Note: Allocate an Elastic IP to your Jenkins-Ansible server.**

STEP 2 – PREPARE YOUR DEVELOPMENT ENVIRONMENT USING VISUAL STUDIO CODE
First part of ‘DevOps’ is ‘Dev’, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable.

- After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.<br>
- Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance<br>
`git clone <ansible-config-mgt repo link><br>`

STEP 3 - BEGIN ANSIBLE DEVELOPMENT
- In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature. <br>
Tip: Give your branches descriptive and comprehensive names, for example, if you use Jira or Trello as a project management tool – include ticket number (e.g. PRJ-145) in the name of your branch and add a topic and a brief description what this branch is about – a bugfix, hotfix, feature, release (e.g. feature/prj-145-lvm) <br>
- Checkout the newly created feature branch to your local machine and start building your code and directory structure
- Create a directory and name it playbooks – it will be used to store all your playbook files.
-- Within the playbooks folder, create your first playbook, and name it common.yml
- Create a directory and name it inventory – it will be used to keep your hosts organised.
- Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

Step 4 – SET UP AN ANSIBLE INVENTORY
An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate.<br>
Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.<br>
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
** Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent:

https://youtu.be/OplGrY74qog

Step 5 – CREATE A COMMON PLAYBOOK
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
        name: wireshark
        state: latest
```

This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers.<br>
It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.<br>

Step 6 – Update GIT with the latest code
Now all of your directories and files live on your machine and you need to push changes made locally to GitHub.
Now you have a separate branch, you will need to know how to raise a Pull Request (PR), get your branch peer reviewed and merged to the master branch.

- Commit your code into GitHub:
- use git commands to add, commit and push your branch to GitHub.
`git status`
`git add <selected files>`
`git commit -m "commit message"`
- Create a Pull request (PR)
- Wear a hat of another developer for a second, and act as a reviewer.
- If the reviewer is happy with your new feature development, merge the code to the master branch.
- Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.<br>

Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server.<br>

Step 7 - RUN FIRST ANSIBLE TEST
Now, it is time to execute ansible-playbook command and verify if your playbook actually works: <br>
`cd ansible-config-mgt`<br>
`ansible-playbook -i inventory/dev.yml playbooks/common.yml`<br>
- You can go to each of the servers and check if wireshark has been installed by running `which wireshark` or `wireshark --version`<br>

** Optional step – Repeat once again **

- Update your ansible playbook with some new Ansible tasks and go through the full checkout -> change codes -> commit -> PR -> merge -> build -> ansible-playbook cycle again to see how easily you can manage a servers fleet of any size with just one command!








