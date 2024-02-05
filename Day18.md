# Ansible

Date: 28-08-2023

## Inventory

- List of servers that Ansible is managing
- It is always recommended to logically group servers based on:
  - Geography -> IN, US, UK, AU, etc
  - Environment -> DEV, QA, PRE-PROD, UAT, PROD
  - Component -> web, app, db
  - Server -> mongodb, cart, catalogue
- For e.g.
  - roboshop-us-dev-db-mongodb-01
  - roboshop-us-dev-db-mongodb-02 (For high availability)
- To check if a DNS record is updated or not, we can use: `nslookup <domain-address>`
  - For e.g. `nslookup mongodb.learninguser.online`

`learning-ansible/inventory`

```inventory
192.168.2.1
172.35.36.96

[mongodb]
roboshop-us-dev-db-mongodb-01.learninguser.online

[mysql]
roboshop-us-dev-db-mysql-01.learninguser.online
roboshop-us-dev-db-mysql-02.learninguser.online

[cart]
roboshop-in-prod-app-cart-01.learninguser.online
roboshop-in-prod-app-cart-02.learninguser.online
roboshop-in-prod-app-cart-03.learninguser.online

[user]
roboshop-in-prod-app-user-01.learninguser.online
roboshop-in-prod-app-user-02.learninguser.online

[db:children]
mongodb
mysql

[app:children]
cart
user

# Sample Inventory File
web1 ansible_host=server1.company.com
web2 ansible_host=server2.company.com
web3 ansible_host=server3.company.com

# Sample Inventory File
  
# Web Servers
web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!

# DB Servers
db1 ansible_host=server4.company.com ansible_connection=winrm ansible_user=administrator ansible_password=Dbp@ss123!

```

- To fetch the hosts that are under a group inside the inventory: `ansible -i inventory mongodb --list-hosts`
- To fetch the hosts that are ungrouped inside the inventory: `ansible -i inventory ungrouped --list-hosts`
- To fetch all the hosts (grouped + ungrouped) inside the inventory: `ansible -i inventory all --list-hosts`
- To prompt for a password when running an ansible adhoc command: `ansible -i inventory mongodb -m ping -u centos --ask-pass`
- Task: Installing NGiNX on the remote server without logging in
  - yum install nginx -y --> command
  - ansible --> module/collection
  - `module/collection name <parameters>`
- To install: `ansible -i inventory mongodb --become -e ansible_user=centos -e ansible_password=DevOps321 -m ansible.builtin.yum -a "name=nginx state=installed"`
  - `--become` or `-b`: To use sudo previliges
  - `-a`: arguments
  - `-e`: extra arguments
- To start the service: `ansible -i inventory mongodb -b -e ansible_user=centos -e ansible_password=DevOps321 -m ansible.builtin.service -a "name=nginx state=started"`
  - yellow color: It successfully changed the configuration on the server
  - Green color: The changes were already made on the server
- Adhoc: Something emergency i.e. to perform a quick check
- Ansible playbooks consists of list of ansible adhoc commands that are written and managed using YAML syntax

## YAML

- Yet Another Markup Language
- Some other Markup languages are:
  - XML: Extensible Markup Language
  - HTML: Hypertext Markup language
- Indentation needs to be proper else it will throw errors
- YAML consists of 3 data types majorly:
  1. Single Key-value pair: E.g. `name: Pavan Kumar`
  2. Map:

      ```yaml
        address:
          city: hyd
          country: india
          dno: 102
          street: ganghi nagar
      ```

  3. List

      ```yaml
        addresses:
        - type: perm
          city: hyd
          country: india
        - type: current
          city: california
          country: US
      ```

## Sample playbook

`https://github.com/sivadevopsdaws74s/ansible/blob/master/01-playbook.yaml`

```yaml
# Playbook is list of plays, always it should start with -
- name: ping the node
  hosts: mongodb
  # this is list of tasks/modules/collections
  tasks:
  - name: pinging the server
    # this is the map
    ansible.builtin.ping:
```

- To run this playbook, we use: `ansible-playbook -i inventory -e ansible_user=centos -e ansible_password=DevOps321 01-playbook.yaml`

## Variables

- Play level variables
- Task level variables
- From files
- From Command prompt
- From inventory file

`https://github.com/sivadevopsdaws74s/ansible/blob/master/03-variables.yaml`

```yaml
- name: variables in ansible
  hosts: mongodb
  # This is Play level variables, map
  vars:
    COURSE: DevOps with AWS
    TRAINER: Sivakumar
    DURATION: 110HRS
  tasks:
  - name: print hello world
    ansible.builtin.debug:
      msg: "Hello, I am learning Ansible"
  - name: print variables
    ansible.builtin.debug:
      msg: "Hello, I am learning {{COURSE}}, Trainer is {{TRAINER}}, Duration is {{DURATION}}"
```
