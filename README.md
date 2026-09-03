Ansible Notes:
1. What is Ansible?
Ansible is an open-source configuration management and automation tool used to manage multiple servers from one central machine.
What we are doing in this practical
We have:
                Ansible Master
                            |
       
       Server_1   Server_2   Server_3
•	Ansible Master → controls/manages the other servers. 
•	Managed Servers → Server 1, Server 2, Server 3. 
•	Ansible connects to the managed servers using SSH. 
•	We can install software, update packages, execute commands, start services, and deploy files from the Ansible Master. 
Important definitions
Terraform: Infrastructure provisioning tool. Used to create infrastructure such as EC2, VPC, subnet, etc.
Ansible: Configuration management and automation tool. Used to configure and manage already-created servers.
Push-based mechanism: Ansible Master pushes commands/configuration to managed servers.
Playbook: A YAML file containing tasks that Ansible executes on one or more servers.
Inventory: A file containing the list of managed servers and their configuration.
Module: A built-in Ansible component used to perform a specific operation, such as apt, copy, service, command, etc.________________________________________Ansible Practical Steps
Step 1: Create Ansible Master Server
AWS → EC2 → Launch Instance → Name: ansible-master → AMI: Ubuntu →
Key Pair: ansible-master-key-pair →Launch the instance.
The Ansible Master is the machine from which we run all Ansible commands.________________________________________Step 2: Create 3 Managed Servers
AWS → EC2 → Launch Instance
Create 3 Ubuntu instances.
For example: server_1, server_2, server_3
Use the required key pair and allow SSH access.
We will use:
server_1 → Managed Server
server_2 → Managed Server
server_3 → Managed Server________________________________________Ansible Installation
Step 3: Connect to Ansible Master
Open PowerShell:
cd C:\Users\<username>\Downloads
Connect to the Ansible Master:
ssh -i "ansible-master-key-pair.pem" ubuntu@<MASTER-PUBLIC-DNS>
After connecting, you should see:
ubuntu@ip-xxxxxxxx:~$
From this point, the commands below are run inside Ubuntu, not Windows PowerShell.________________________________________Step 4: Install Ansible
Add Ansible repository:
sudo apt-add-repository ppa:ansible/ansible
Update packages:
sudo apt update
Install Ansible:
sudo apt install ansible
Check Ansible version:
ansible --version
What we did
We installed Ansible on the Ansible Master.________________________________________Ansible Inventory
Step 5: Create/Update Hosts File
Ansible inventory file:
/etc/ansible/hosts
Open it:
sudo nano /etc/ansible/hosts
or:
sudo vim /etc/ansible/hosts
Inventory configuration
Use your actual public IPs:
[servers]
server_1 ansible_host=<SERVER_1_PUBLIC_IP>
server_2 ansible_host=<SERVER_2_PUBLIC_IP>

[prod]
server_3 ansible_host=<SERVER_3_PUBLIC_IP>

[servers:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/keys/ansible-master-key-pair.pem

If your actual key file is named ansible-master-key-demo.pem, use that exact filename instead.
Meaning
[servers]
Creates a group called servers.
server_1 ansible_host=<IP>
Defines Server 1 and its IP address.
[prod]
Creates a production group.
server_3 ansible_host=<IP>
Puts Server 3 into the prod group.________________________________________Copy SSH Private Key to Ansible Master
Step 6: Create Keys Directory
The private key needs to be available on the Ansible Master.
From the Ansible Master:
mkdir -p /home/ubuntu/keys
From Windows PowerShell, copy the key:
scp -i "ansible-master-key-pair.pem" ansible-master-key-pair.pem ubuntu@<MASTER-PUBLIC-DNS>:/home/ubuntu/keys/
Check the file on Ubuntu:
ls -l /home/ubuntu/keys/________________________________________Step 7: Set Private Key Permission
On the Ubuntu Ansible Master, run:
chmod 400 /home/ubuntu/keys/ansible-master-key-pair.pem
Check:
ls -l /home/ubuntu/keys/
The permission should be similar to:
-r--------
Why chmod 400?
SSH private keys must not be accessible by other users.________________________________________SSH Host-Key Configuration
Step 8: Disable Host Key Checking
Open:
sudo nano /etc/ansible/ansible.cfg
Add:
[defaults]
host_key_checking = False
Save:
Ctrl + O
Enter
Ctrl + X
Why?
This prevents Ansible from stopping because of SSH host-key verification when connecting to the managed servers.________________________________________Test Ansible Connection
Step 9: Test All Servers
Run:
ansible servers -m ping
Meaning
ansible
Run Ansible.
servers
Use the [servers] inventory group.
-m ping
Use the Ansible ping module to test connectivity.
Expected:
server_1 | SUCCESS
"ping": "pong"

server_2 | SUCCESS
"ping": "pong"________________________________________Ansible Inventory Verification
Step 10: Check Inventory
ansible-inventory --list
To see hosts in a particular group:
ansible servers --list-hosts
For production:
ansible prod --list-hosts________________________________________Execute Linux Commands Using Ansible
Step 11: Check RAM
ansible servers -a "free -h"
Meaning
ansible
Run Ansible.
servers
Run against the servers group.
-a
--args → specifies the command to execute.
free -h
Linux command used to display memory information.
-h means human-readable.
Example:
Mem:
total   used   free
1.9Gi   300Mi  1.2Gi
The total value shows the server's RAM.________________________________________Step 12: Update All Servers
ansible servers -a "sudo apt update"
This executes:
sudo apt update
on all servers in the servers group.________________________________________Ansible Playbook
Step 13: Create Playbooks Directory
cd ~
mkdir playbooks
cd playbooks
Check:
ls________________________________________Date Playbook
Step 14: Create Date Playbook
Create:
vim date_play.yml
Enter:
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
It runs the date command on all servers in the servers group.________________________________________Nginx Installation Playbook
Step 15: Create Nginx Playbook
vim install_nginx_play.yml
Enter:
- name: Install nginx and start it
  hosts: servers
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: latest

    - name: Start nginx
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
1.	Installs Nginx. 
2.	Starts Nginx. 
3.	Enables Nginx so it starts automatically after reboot. 
Important
Because:
hosts: servers
this playbook runs on:
server_1
server_2
It does not run on server_3 if server_3 is only in [prod].________________________________________Static Website Deployment
Step 16: Create HTML Page
Go to:
cd ~/playbooks
Create:
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
Check:
ls
You should have:
date_play.yml
install_nginx_play.yml
index.html________________________________________Production Group
Step 17: Update Inventory
We changed the inventory to separate production servers:
[servers]
server_1 ansible_host=<SERVER_1_PUBLIC_IP>
server_2 ansible_host=<SERVER_2_PUBLIC_IP>

[prod]
server_3 ansible_host=<SERVER_3_PUBLIC_IP>

[servers:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/keys/ansible-master-key-pair.pem
Check:
ansible-inventory --list
Check production server:
ansible prod --list-hosts
Expected:
hosts (1):
    server_3________________________________________Static Website Playbook
Step 18: Create Static Website Playbook
vim deploy_static_page_play.yml
Enter:
- name: Install nginx and server static website
  hosts: prod
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: latest

    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Deploy web page
      copy:
        src: index.html
        dest: /var/www/html/
Save________________________________________Step 19: Run Static Website Playbook
ansible-playbook deploy_static_page_play.yml
What happens?
Because:
hosts: prod
Ansible runs the playbook on the servers inside [prod].
In our inventory:
[prod]
server_3
So:
Ansible Master
      ↓
   server_3
      |
      ├── Install Nginx
      ├── Start Nginx
      └── Copy index.html
The file is copied to:
/var/www/html/
This is the default Nginx web root on Ubuntu.________________________________________Verify Nginx
Step 20: Check Nginx Remotely
Instead of checking Nginx on the Ansible Master, check the managed server:
ansible prod -a "systemctl status nginx --no-pager"
Or:
ansible prod -a "systemctl is-active nginx"
Expected:
active________________________________________Verify Website
Step 21: Test Website From Server
Run:
ansible prod -a "curl -I http://localhost"
Expected:
HTTP/1.1 200 OK
You can also check the HTML:
ansible prod -a "curl http://localhost"________________________________________Access Website From Chrome
Step 22: Open Website
AWS → EC2 → Instances → Select the production server.
Copy its Public IPv4 address.
Open Chrome:
http://<PUBLIC-IP>
Example:
http://54.xx.xx.xx
AWS Security Group
Make sure the server's Security Group allows:
HTTP
TCP
Port 80
Source: 0.0.0.0/0
Otherwise Nginx may be running correctly but the website will not be accessible from Chrome.________________________________________Important Paths Used
Path	Purpose
/etc/ansible/hosts	Ansible inventory file
/etc/ansible/ansible.cfg	Ansible configuration
/home/ubuntu/keys/	Location of private SSH key
/home/ubuntu/keys/ansible-master-key-pair.pem	SSH private key
~/playbooks/	Playbooks directory
~/playbooks/index.html	Website source file
~/playbooks/date_play.yml	Date playbook
~/playbooks/install_nginx_play.yml	Nginx installation playbook
~/playbooks/deploy_static_page_play.yml	Static website deployment playbook
/var/www/html/	Nginx web root
________________________________________
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
14. Install/start Nginx
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
