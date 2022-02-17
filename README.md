# ansible

# Introduction
## why
    To perform a repititive tasks
    Scripts Vs Ansible PlayBook
    Time, code, .... vs Ansible.
    Usecase:
        1. Simple: Restart hosts in a order which is WebServer, Database, etc,..
        2. Complex: Provision 100s of VMs in public / private cloud
            Easily integrate with rest of environment so that you can pull this information to be used in the automation process, such as get data from a CMDB database, to get list of VMs, to target, to trigger automation process when service now workflow approves etc,..
        3. Ansible documentation: https://docs.ansible.com/ansible_community.html, https://docs.ansible.com/ansible/latest/index.html
# Setting up Ansible on VB
    1. Ansible Control machine.
    2. Ansible Target Machine 01
    3. Ansible Target Machine 02

    Username: osboxes
Password: osboxes.org
Root Account Password: osboxes.org
Guest Tools: Open-VM-Tools Installed
Keyboard Layout: US (Qwerty)
VMware Compatibility: Version 10+

CentOS 8

Kernel driver not installed (rc=-1908)

Make sure the kernel module has been loaded successfully.

where: suplibOsInit what: 3 VERR_VM_DRIVER_NOT_INSTALLED (-1908) - The support driver is not installed. On linux, open returned ENOENT. 


# Introduction to YAML
Ansible playbooks are text files or configuration files that are written in a particular format called YAML

# Inventory Files
## inventory.txt
target1 ansible_host=192.168.20.91 ansible_ssh_pass-osboxes.org
target2 ansible_host=192.168.20.94 ansible_ssh_pass-osboxes.org

## Another Sample Inventory file
# Sample Inventory File

# Web Servers
sql_db1 ansible_host=sql01.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass
sql_db2 ansible_host=sql02.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass
web_node1 ansible_host=web01.xyz.com ansible_connection=ssh ansible_user=administrator ansible_ssh_pass=Win$Pass
web_node2 ansible_host=web02.xyz.com ansible_connection=ssh ansible_user=administrator ansible_ssh_pass=Win$Pass
web_node3 ansible_host=web03.xyz.com ansible_connection=ssh ansible_user=administrator ansible_ssh_pass=Win$Pass

[db_nodes]
sql_db1
sql_db2

[web_nodes]
web_node1
web_node2
web_node3

[boston_nodes]
sql_db1
web_node1

[dallas_nodes]
sql_db2
web_node2
web_node3

[us_nodes:children]
boston_nodes
dallas_nodes

# Playbooks
## ansible commands:
ansible <hosts> -a <command>
ansible all -a "/sbin/reboot"
ansible <hosts> -m <module>
ansible target1 -m ping

ansible all -m ping -i inventory.txt

## ansible-playbook commands:
ansible-playbook <playbook-name>
ansible-playbook playbook-webserver.yaml

playbook-pingtest.yaml
- 
  name: Test connectivity to target servers
  hosts: all
  tasks: 
    - name: Ping test
      ping: 

ansible-playbook playbook-pingtest.yaml -i inventory.txt

## Validate YAML format
www.yamlint.com

# Modules
## System
## Commands
- Command
- Expect
- Raw
- Script
- Shell
### Example playbook.yml
-
  name: Play 1
  hosts: localhost
  tasks:
    - name: Execute Command 'date'
      command: date
    - name: Create folder
      command: mkdir /folder creates=/folder

### Example Script playbook.yml
1. copy the script to remote systems
2. Execute the script on remote systems
-
  name: Play 1
  hosts: localhost
  tasks:
    - name: Execute Command 'date'
      command: date
    - name: Run a script on remote server
      script: /some/local/script.sh -arg1 -arg2

## Files
- Acl, Archive, Copy, File, Find, Lineinfile, Replace, Stat, Template, Unarchive
### Example lineinfile playbook.yml
This is to add DNS for differene environment or similar

## Databsae
## Cloud
- Amazon, Atomic, Azure, Centry link, Cloudscale,... Docker,..
## Windows
- Create WebSite in IIS
## More..

### Example:-
-
    name: 'Execute a script on all web server nodes and start httpd service'
    hosts: web_nodes
    tasks:
        -
            name: 'Update entry into /etc/resolv.conf'
            lineinfile:
                path: /etc/resolv.conf
                line: 'nameserver 10.1.250.10'
        -
            name: 'Add the user'
            user:
                
                uid: 1040
                group: developers
        -
            name: 'Execute a script'
            script: /tmp/install_script.sh
        -
            name: 'Start httpd service'
            service:
                name: httpd
                state: present

### Example 2:-
#### Without Variable:-
-
            name: 'Add DNS Server'
            hosts: localhost
            vars:
                 dns_server: 10.1.250.10
            tasks:
                - lineinfile
                        path: /etc/resolv.conf
                        line: 'nameserver 10.1.250.10'

#### With Variable:-

-
            name: 'Add DNS Server'
            hosts: localhost
            vars:
                 dns_server: 10.1.250.10
            tasks:
                - lineinfile
                        path: /etc/resolv.conf
                        line: 'nameserver {{ }}'

# Variables
# Conditionals
## Conditions


when: ansible_os_family == "Debian" and ansible_distribution_version == "16.04"

when: ansible_os_family == "Redhat" or ansible_os_family == "SUSE"

### Example 1:-
-
    name: 'Execute a script on all web server nodes'
    hosts: all_servers
    tasks:
        -
            service: 'name=mysql state=started'
            when: 'ansible_host == "server4.company.com"'

### Example 2:-
-
    name: 'Am I an Adult or a Child?'
    hosts: localhost
    vars:
        age: 25
    tasks:
        -
            command: 'echo "I am a Child"'
            when: 'age < 18'
        -
            command: 'echo "I am an Adult"'
            when: 'age >= 18'

### Example 3:-
-
    name: 'Add name server entry if not already entered'
    hosts: localhost
    tasks:
        -
            register: "command_output"
            shell: 'cat /etc/resolv.conf'
        -
            shell: 'echo "nameserver 10.0.250.10" >> /etc/resolv.conf'
            when: 'command_output.stdout.find("10.0.250.10") == -1'

### Conditions with Loop

# Loops
1. item

Example 1:-
-
    name: 'Print list of fruits'
    hosts: localhost
    vars:
        fruits:
            - Apple
            - Banana
            - Grapes
            - Orange
    tasks:
        -
            command: 'echo "{{ item }}"'
            with_items: "{{ fruits }}"


2. item.name

Example 2:-
-
    name: 'Install required packages'
    hosts: localhost
    vars:
        packages:
            - httpd
            - binutils
            - glibc
            - ksh
            - libaio
            - libXext
            - gcc
            - make
            - sysstat
            - unixODBC
            - mongodb
            - nodejs
            - grunt
    tasks:
        -
            yum: 'name={{ item }} state=present'
            with_items: "{{ packages }}"


3. with_<<lookup plugins>>

#### Lookup Plugins
Just think of them as custom scripts that can do a specific task like read files, connect to urls, connect to database

with_items
with_url
with_mongodb
with_...
..
.

with_k8s
with_openshift



# Roles

Assigning a role to a server ex: mysql, webserver,.. Similar to Doctor, Engineer role, etc,..
Role means everything you need to do to make a server a database server.
Ex: 
installing prerequistie for Mysql
installine mysql database 
configuring the mysql service
configuring database and users
...

Similar to say to become a Dr Role 
Going to medical school
Earn medical degree
Complete residency program
Obtain license

Primary purpose of role is to reuse for other tasks or projects within your organisation or outside others globally

Roles also help in organizing your code within the ansible. Roles introduce a set of best practices that must be followed to organize:
- task: all tasks into a task directory, 
- vars: all variables used by the tasks goes into the vars directory, 
- defaults: any default values goes in the default directory, 
- handlers: all handlers goes into the handlers directory, 
- templates: and any template used by playbooks goes in the template directory

#### Sharing
Roles also help in sharing your code with others in the ansible community.
One such community: Ansible Galaxy cotains 1000 of roles for almost any task you can think off

#### How to create a directory Structure
This is to create a own role from the scratch, then move all of your code to tasks directory in your file.

command:
ansible-galaxy init mysql

output:
mysql
|-- Readme.md
|-- templates
|-- tasks
|-- handlers
|-- vars
|-- defaults
--

#### How to use your role within your playbook

Directory:
my-playbook
|-- playbook.yml
--    

playbook.yml:
- name: Install and Configure MySQL
   hosts: db-server
   roles: 
    - mysql

#### How can my playbook find the role

1. One way:
my-playbook
|-- playbook.yml
|-- roles
    |-- mysql
        |-- Readme.md
        |-- templates
        |-- tasks
        |-- handlers
        |-- vars
        |-- defaults
        --

2. You can move roles to common directory designated for a system at /etc/ansible/roles location. That is the default location where ansible searches for roles; if it cant be found in a playbook directory Ofcourse that is defined in the ansible configuration file /etc/ansible/ansible.cfg
roles_path = /etc/ansible/roles

#### Sharing Roles

Once you have created Roles and used it in your playbook, you can share it through ansible galaxy github repository

#### Find roles in ansible galaxy ui

Find:
1. UI
2. command line: ansible-galaxy search mysql

Install:
ansible-galaxy install <<name-of-the-role>> 
ex: ansible-galaxy install geerlingguy.mysql

The role is extracted to the default role directory: /etc/ansible/roles/geerlingguy.mysql


Use:
1. Array 

- name: Install and Configure MySQL
   hosts: db-server
   roles: 
    - geerlingguy.mysql

2. Dictionary

- name: Install and Configure MySQL
   hosts: db-server
   roles: 
    - role: geerlingguy.mysql
      become: yes
      vars:
        mysql_user_name: db_user

#### List Roles
ansible galaxy list
- geerlingguy.mysql
- kodecloud1.mysql

#### View the location where roles will be installed
ansible-config dump | grep ROLE
You can see default roles path where the roles are installed

#### How to Install roles
ansible-galaxy install geerlingguy.mysql -p ./roles
While installing roles you may use the -p option to install it in the current directory under roles like above command


#### Summarize

Roles make easy to develop, reuse and share playbooks

ex:

- name: Install and Configure MySQL
   hosts: db-and-webserver
   roles: 
    - geerlingguy.mysql
    - nginx


# Advances Topics

## Preparing Windows Server
docs.ansible.com -> Windows support

https://docs.ansible.com/ansible/latest/user_guide/windows.html#windows
https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html#authentication-options

## Ansible-Galaxy
galaxy.ansible.com
-> browse roles

## Patterns
You can see hosts: localhost in playbook
Additionally you can also have: 
- host1, host2, host3
- ..
- .
- *.company.com
docs.ansible.com pattern

## Dynamic Inventory
ansible-playbook -i inventory.txt playbook.yml
ansible-playbook -i inventory.py playbook.yml

## Developing Custom Modules
Create your own Module (Plugin)


# Demo Project
kodekloudhub/learning-app-ecommerce

LAMP - Linux, Ansible, MariaDB, PHP
Install MariaDB

## Practice, Practice, Practice

# Advanced
## Roles
## Asynchronous Actions
## Error Handling
## Jinja 2 Templating
## Lookups 
## Vault
## Dynamic Inventory
## Custom Modules
## Plugins
## Practice, Practice, Practice
