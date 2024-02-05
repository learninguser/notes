# Ansible: Variables, Datatypes and Conditions

Date: 29-08-2023

## Variables

### Vars from files

`ansible/04-vars-files.yaml`

```yaml
- name: variables from files
  hosts: localhost #managing the ansible server itself
  vars_files:
  - variables.yaml
  tasks:
  - name: printing variables
    ansible.builtin.debug:
      msg: "We are learning {{NAME}}, Duration is: {{DURATION}}, Trainer is: {{TRAINER}}"
```

`ansible/variables.yaml`

```yaml
NAME: DevOps with AWS
DURATION: 120HRS
TRAINER: Sivakumar
```

### Values to Vars during runtime

`ansible/05-vars-prompt.yaml`

```yaml
- name: variables from prompt
  hosts: localhost
  vars_prompt:
  - name: USERNAME
    prompt: Please enter your username
    private: false # you can see the value entered on screen
  - name: PASSWORD
    prompt: Please enter your password
    private: true # you can't see the value entered on screen
  tasks:
  - name: print variable values
    ansible.builtin.debug:
      msg: "username: {{USERNAME}}, password: {{PASSWORD}}"
```

### Task level variables

- Can inherit the variables from play level
- Task level can extend and override these variables

`ansible/06-task-level.yaml`

```yaml
- name: variables at task level
  hosts: localhost
  # these variables of parent or play level
  vars:
  - money: "10000 RS"
    land: "100 hectars"
  tasks:
  - name: inherit values from play
    ansible.builtin.debug:
      msg: "MONEY: {{money}}, LAND: {{land}}"
  - name: inherit values from play and add and override
    vars:
    - money: "200000 RS"
      houses: "3 houses"
    ansible.builtin.debug:
      msg: "MONEY: {{money}}, LAND: {{land}}, houses: {{houses}}"
```

- To run the playbook: `ansible-playbook 06-task-level-yaml`

### From inventory file

`ansible/07-inventory.yaml`

```yaml
- name: variables from inventory
  hosts: mongodb
  tasks:
  - name: print mongodb username
    ansible.builtin.debug:
     msg: "username is: {{MONGO_USERNAME}}"
```

`ansible/inventory`

```ìnventory
[mongodb]
172.31.42.158

[mongodb:vars]
MONGO_USERNAME=mongodbadmin
MONGO_DB=categories
```

- To run this playbook, we use: `ansible-playbook -i inventory -e ansible_user=centos -e ansible_password=DevOps321 01-playbook.yaml`

### From command line

`ansible/08-command_line.yaml`

```yaml
- name: variables from command line
  hosts: localhost
  tasks:
  - name: print variable from command line
    ansible.builtin.debug:
    msg: "The value of variable course is {{COURSE}}"
```

- To run this playbook, we use: `ansible-playbook -i inventory -e ansible_user=centos -e ansible_password=DevOps321 -e COURSE="DevOps with AWS" 08-command_line.yaml`

### Variable Precedence

- 1st preference is command line args
- 2nd preference is task level
- 3rd preference is from vars_files
- 4th preference is from prompt
- 5th preference is from play level
- 6th preference is from inventory

## Data Types

1. Number
2. String
3. List
4. Map
5. Boolean

`ansible/data_types.yaml`

```yaml
- name: ansible variable data types 
  hosts: localhost
  vars:
    - AGE: 30 # Number
    - NAME: "Sivakumar" # String
    - isDevOps: true # Boolean
    - Skills: # List
      - DevOps
      - AWS
      - Docker
    - EXPERIENCE: # Map
        DevOps: 7
        AWS: 5
        Docker: 4
  tasks:
    - name: print number variable
      ansible.builtin.debug:
        msg: "{{AGE}}"
    - name: print String variable
      ansible.builtin.debug:
        msg: "{{NAME}}"
    - name: print Boolean variable
      ansible.builtin.debug:
        msg: "{{isDevOps}}"
    - name: print List variable
      ansible.builtin.debug:
        msg: "{{Skills}}"
    - name: print Map variable
      ansible.builtin.debug:
        msg: "{{EXPERIENCE}}"
```

## Conditions

`ansible/simple_condition.yaml`

```yaml
- name: simple condition
  hosts: localhost
  vars:
    NAME: DevOps1
  tasks:
  - name: run this if name is DevOps
    ansible.builtin.debug:
      msg: "Hello {{NAME}}"
    when: NAME == "DevOps"
```

- We can register variables to store the output of a command
- when we dont have an equivalent module for some linux commands, we use `ansible.builtin.shell` or `ansible.builtin.command`
- when there is a failure at the time of command execution, ansible stops the execution of playbook at that task itself
- We can also ignore the erros by setting: `ignore_errors: true`

```yaml
- name: create user
  hosts: localhost
  tasks:
    - name: check roboshop user exist or not
      ansible.builtin.command: id roboshop
      register: out # out is variable name
      ignore_errors: true
      
    - name: print exit status
      ansible.builtin.debug:
        msg: "{{out.rc}}"
    
    - name: create user roboshop
      become: yes # sudo access for this task only
      ansible.builtin.user:
        name: roboshop
      when: out.rc != 0
```
