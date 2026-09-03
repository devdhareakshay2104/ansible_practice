Ansible Practical

This repository contains my Ansible learning and practice work.
The practical covers Ansible installation, inventory configuration, SSH connectivity, ad-hoc commands, playbooks, Nginx installation, and static website deployment.

What is Ansible?

Ansible is an open-source configuration management and automation tool used to manage multiple servers from one central machine.

In this practical:

Ansible Master → Controls and manages the other servers.

Managed Servers → Server 1, Server 2, and Server 3.

Ansible connects to managed servers using SSH.

Ansible can install software, update packages, execute commands, start services, and deploy files.

Important Definitions

Term

Definition

Terraform

Infrastructure provisioning tool used to create infrastructure such as EC2, VPC, and subnets.

Ansible

Configuration management and automation tool used to configure and manage already-created servers.

Push-based mechanism

Ansible Master pushes commands/configuration to managed servers.

Playbook

A YAML file containing tasks that Ansible executes on one or more servers.

Inventory

A file containing the list of managed servers and their configuration.

Module

A built-in Ansible component used to perform a specific operation, such as apt, copy, service, or command.

Host

A remote server or machine managed by Ansible.

Host Group

A collection of hosts that can be managed together.

Ansible Practical Steps

Step 1: Create Ansible Master Server

AWS → EC2 → Launch Instance

Name: ansible-master

AMI: Ubuntu

Key Pair: ansible-master-key-pair

Launch the instance.

The Ansible Master is the machine from which all Ansible commands are executed.

Step 2: Create 3 Managed Servers

AWS → EC2 → Launch Instance

Create three Ubuntu instances:

server_1

server_2

server_3

Use the required key pair and allow SSH access.

These servers will be managed by the Ansible Master.

Ansible Installation

Step 3: Connect to Ansible Master

Open Windows PowerShell:

cd C:\Users\<username>\Downloads

Connect to the Ansible Master:

ssh -i "ansible-master-key-pair.pem" ubuntu@<MASTER-PUBLIC-DNS>

After connecting, you should see something similar to:

ubuntu@ip-xxxxxxxx:~$

From this point onward, the Ansible commands are run inside Ubuntu, not Windows PowerShell.

Step 4: Install Ansible

Add the Ansible repository:

sudo apt-add-repository ppa:ansible/ansible

Update packages:

sudo apt update

Install Ansible:

sudo apt install ansible

Check the installed version:

ansible --version

What we did

We installed Ansible on the Ansible Master.

Ansible Inventory

Step 5: Create / Update Hosts File

The Ansible inventory file is:

/etc/ansible/hosts

Open the inventory file:

sudo nano /etc/ansible/hosts

or:

sudo vim /etc/ansible/hosts

Use your actual public IP addresses:

[servers]
server_1 ansible_host=<SERVER_1_PUBLIC_IP>
server_2 ansible_host=<SERVER_2_PUBLIC_IP>

[prod]
server_3 ansible_host=<SERVER_3_PUBLIC_IP>

[servers:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/keys/ansible-master-key-pair.pem

If your key file has a different name, use the exact filename.

Inventory Explanation

[servers]

Creates a group named servers.

server_1 ansible_host=<IP>

Defines Server 1 and its IP address.

[prod]

Creates a production group.

server_3 ansible_host=<IP>

Places Server 3 into the prod group.

SSH Private Key Setup

Step 6: Create Keys Directory

On the Ansible Master:

mkdir -p /home/ubuntu/keys

From Windows PowerShell, copy the private key to the Ansible Master:

scp -i "ansible-master-key-pair.pem" ansible-master-key-pair.pem ubuntu@<MASTER-PUBLIC-DNS>:/home/ubuntu/keys/

Check the key on Ubuntu:

ls -l /home/ubuntu/keys/

Step 7: Set Private Key Permission

On the Ansible Master:

chmod 400 /home/ubuntu/keys/ansible-master-key-pair.pem

Check the permission:

ls -l /home/ubuntu/keys/

The permission should be similar to:

-r--------

Why chmod 400?

SSH private keys must not be accessible by other users.

SSH Host-Key Configuration

Step 8: Disable Host Key Checking

Open the Ansible configuration file:

sudo nano /etc/ansible/ansible.cfg

Add:

[defaults]
host_key_checking = False

Save the file.

Why?

This prevents Ansible from stopping because of SSH host-key verification when connecting to managed servers.

Test Ansible Connection

Step 9: Test All Servers

Run:

ansible servers -m ping

Command Explanation

ansible → Runs Ansible.

servers → Uses the [servers] inventory group.

-m ping → Uses the Ansible ping module to test connectivity.

Expected result:

server_1 | SUCCESS
"ping": "pong"

server_2 | SUCCESS
"ping": "pong"

Ansible Inventory Verification

Step 10: Check Inventory

Display the inventory:

ansible-inventory --list

List hosts in the servers group:

ansible servers --list-hosts

List hosts in the production group:

ansible prod --list-hosts

Execute Linux Commands Using Ansible

Step 11: Check RAM

Run:

ansible servers -a "free -h"

Command Explanation

ansible → Runs Ansible.

servers → Runs against the servers group.

-a / --args → Specifies the command to execute.

free -h → Linux command used to display memory information.

-h → Displays the output in human-readable format.

Example:

Mem:
total   used   free
1.9Gi   300Mi  1.2Gi

The total value shows the server's RAM.

Step 12: Update All Servers

Run:

ansible servers -a "sudo apt update"

This executes:

sudo apt update

on all servers in the servers group.

Ansible Playbooks

Step 13: Create Playbooks Directory

Create a directory for playbooks:

cd ~
mkdir playbooks
cd playbooks

Check the directory:

ls

Date Playbook

Step 14: Create Date Playbook

Create the playbook:

vim date_play.yml

Add:

- name: Dates Playbook
  hosts: servers
  tasks:
    - name: Show date
      command: date

    - name: Show date again
      command: date

Save:

:wq

Run:

ansible-playbook date_play.yml

What this does

It runs the date command on all servers in the servers group.

Nginx Installation Playbook

Step 15: Create Nginx Playbook

Create:

vim install_nginx_play.yml

Add:

- name: Install nginx and start it
  hosts: servers
  become: yes
  tasks:

    - name: Install Nginx
      apt:
        name: nginx
        state: latest

    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: yes

Save:

:wq

Run:

ansible-playbook install_nginx_play.yml

What we did

The playbook:

Installs Nginx.

Starts Nginx.

Enables Nginx so it starts automatically after reboot.

Because:

hosts: servers

the playbook runs on:

server_1

server_2

Static Website Deployment

Step 16: Create HTML Page

Go to the playbooks directory:

cd ~/playbooks

Create the HTML file:

vim index.html

Example:

<!DOCTYPE html>
<html>
<head>
    <title>My Ansible Website</title>
</head>
<body>
    <h1>Hello from Ansible</h1>
    <h2>Static Website Deployment</h2>
</body>
</html>

Save:

:wq

Check the files:

ls

Expected files:

date_play.yml
install_nginx_play.yml
index.html

Production Group

Step 17: Update Inventory

The inventory is separated into servers and prod groups:

[servers]
server_1 ansible_host=<SERVER_1_PUBLIC_IP>
server_2 ansible_host=<SERVER_2_PUBLIC_IP>

[prod]
server_3 ansible_host=<SERVER_3_PUBLIC_IP>

[servers:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/keys/ansible-master-key-pair.pem

Check the inventory:

ansible-inventory --list

Check the production server:

ansible prod --list-hosts

Expected:

hosts (1):
    server_3

Static Website Playbook

Step 18: Create Static Website Playbook

Create:

vim deploy_static_page_play.yml

Add:

- name: Install nginx and server static website
  hosts: prod
  become: yes
  tasks:

    - name: Install Nginx
      apt:
        name: nginx
        state: latest

    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Deploy web page
      copy:
        src: index.html
        dest: /var/www/html/

Save the file.

Step 19: Run Static Website Playbook

Run:

ansible-playbook deploy_static_page_play.yml

What happens?

Because:

hosts: prod

Ansible runs the playbook on the servers inside the [prod] group.

In our inventory:

[prod]
server_3

So the flow is:

Ansible Master
      |
      v
   server_3
      |
      +-- Install Nginx
      |
      +-- Start Nginx
      |
      +-- Copy index.html

The HTML file is copied to:

/var/www/html/

This is the default Nginx web root on Ubuntu.

Verify Nginx

Step 20: Check Nginx Remotely

Check Nginx status:

ansible prod -a "systemctl status nginx --no-pager"

Or:

ansible prod -a "systemctl is-active nginx"

Expected:

active

Verify Website

Step 21: Test Website From Server

Check the HTTP response:

ansible prod -a "curl -I http://localhost"

Expected:

HTTP/1.1 200 OK

You can also check the HTML:

ansible prod -a "curl http://localhost"

Access Website From Chrome

Step 22: Open Website

AWS → EC2 → Instances → Select the production server.

Copy its Public IPv4 address.

Open Chrome:

http://<PUBLIC-IP>

Example:

http://54.xx.xx.xx

AWS Security Group

Make sure the production server's Security Group allows:

Type

Protocol

Port

Source

HTTP

TCP

80

0.0.0.0/0

Otherwise, Nginx may be running correctly but the website will not be accessible from Chrome.

Important Paths

Path

Purpose

/etc/ansible/hosts

Ansible inventory file

/etc/ansible/ansible.cfg

Ansible configuration file

/home/ubuntu/keys/

Location of private SSH key

/home/ubuntu/keys/ansible-master-key-pair.pem

SSH private key

~/playbooks/

Playbooks directory

~/playbooks/index.html

Website source file

~/playbooks/date_play.yml

Date playbook

~/playbooks/install_nginx_play.yml

Nginx installation playbook

~/playbooks/deploy_static_page_play.yml

Static website deployment playbook

/var/www/html/

Nginx web root

Final Practical Flow

1. Create Ansible Master
        ↓
2. Create Server 1, Server 2, Server 3
        ↓
3. SSH into Ansible Master
        ↓
4. Install Ansible
        ↓
5. Configure /etc/ansible/hosts
        ↓
6. Copy SSH private key to /home/ubuntu/keys/
        ↓
7. chmod 400 private key
        ↓
8. Configure /etc/ansible/ansible.cfg
        ↓
9. ansible servers -m ping
        ↓
10. Check RAM using free -h
        ↓
11. Create ~/playbooks/
        ↓
12. Create date_play.yml
        ↓
13. Create install_nginx_play.yml
        ↓
14. Install and start Nginx
        ↓
15. Create index.html
        ↓
16. Create deploy_static_page_play.yml
        ↓
17. Deploy index.html to /var/www/html/
        ↓
18. Check Nginx
        ↓
19. Open Public IP in Chrome
        ↓
20. Static website displayed

Learning Outcome

Through this practical, I learned:

Ansible Master and Managed Servers

Ansible Inventory

Hosts and Host Groups

SSH-based connectivity

Ansible ad-hoc commands

Ansible modules

YAML Playbooks

Installing and managing Nginx

Deploying a static website using Ansible

Verifying services remotely

Accessing a deployed website through an EC2 Public IP
