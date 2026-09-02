ansible practice :
here created server1,2,3
but server3 is prod
# Ex 2: A collection of hosts belonging to the 'webservers' group:

[servers]
server_1 ansible_host=107.21.15.13
server_2 ansible_host=54.205.30.95

[prod]
server_3 ansible_host=54.196.3.11

[servers:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/keys/ansible-master-key-pair.pem

