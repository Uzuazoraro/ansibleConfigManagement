
A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. If you think about the current architecture you are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provides better security and reduces attack surface.

Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses

## Task

• Install and configure Ansible client to act as a Jump Server/Bastion Host
• Create a simple Ansible playbook to automate servers configuration

## INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

1. Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.
2. In your GitHub account create a new repository and name it ansible-config-mgt.
3. Install Ansible

`sudo apt update`
`sudo apt install ansible`

Check your Ansible version by running `ansible --version`
4. Configure Jenkins build job to save your repository content every time you change it – this will solidify your Jenkins configuration skills acquired in Project 9.
• Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.
• Configure Webhook in GitHub and set webhook to trigger ansible build.
• Configure a Post-build job to save all (**) files, like you did it in Project 9.
5. Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder
`ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`

## Step 2 – Prepare your development environment using Visual Studio Code

1. First part of ‘DevOps’ is ‘Dev’, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable – you need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs – Visual Studio Code (VSC), you can get it here.
2. After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.
3. Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance
git clone <ansible-config-mgt repo link>

## BEGIN ANSIBLE DEVELOPMENT

1. In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.
Tip: Give your branches descriptive and comprehensive names, for example, if you use Jira (<https://www.atlassian.com/software/jira>) or Trello (<https://trello.com/>) as a project management tool – include ticket number (e.g. PRJ-145) in the name of your branch and add a topic and a brief description what this branch is about – a bugfix, hotfix, feature, release (e.g. feature/prj-145-lvm)
2. Checkout the newly created feature branch to your local machine and start building your code and directory structure
3. Create a directory and name it playbooks – it will be used to store all your playbook files.
`sudo mkdir playbooks`
4. Create a directory and name it inventory – it will be used to keep your hosts organised.
`sudo mkdir inventory`
5. Within the playbooks folder, create your first playbook, and name it common.yml
`touch common.yml`
6. Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.
`touch dev`
`touch staging`
`touch uat`
`touch prod`

## Step 4 – Set up an Ansible Inventory

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.
Save below inventory structure in the inventory/dev file to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.
Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent:
To learn how to setup SSH agent and connect VS Code to your Jenkins-Ansible instance, please see this video:
•For Windows users – ssh-agent on windows
•For Linux users – ssh-agent on linux
Steps:
`ssh-agent`
`eval`ssh-agent -s``
`cd downloads`
`ssh-add <path-to-private-key>`
OR
`ssh-add -k <EC2 Instance key>`
Confirm the key has been added with the command below, you should see the name of your key
`ssh-add -l`
Now, ssh into your Jenkins-Ansible server using ssh-agent
`ssh -A ubuntu@public-ip`

## Update your inventory/dev.yml file with this snippet of code

[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user'

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'

## Step 5 – Create a Common Playbook

It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in inventory/dev.
In common.yml playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update your playbooks/common.yml file with following code
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
