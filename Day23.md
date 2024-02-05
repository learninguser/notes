# Miscelleneous topics in Ansible

Date: 05-09-2023

- If we check `ansible --version`, we can get the path to the Ansible configuration and it is present at `/etc/ansible/ansible.cfg`
- Rather than passing the username and password inorder to connect to the server using command line, we can use configuration file `ansible.cfg` to serve this purpose
- To define an environment variable that is valid for a particular session, we use: `export`
- To remove an environment variable, we use: `unset` command

  ```bash
  export a=10
  unset a
  ```

## Configuration precedence in Ansible

1. ANSIBLE_CONFIG -> ENV variable using: `export ANSIBLE_CONFIG=/home/centos/test/ansible.cfg`
2. Current working directory
3. User home directory, it should be `.ansible.cfg`
4. /etc/ansible/ansible.cfg

- Tags in Ansible can be used at any level i.e. Task level, play level etc
- Tags are useful when re-running the playbook for e.g. when a  new release of the artifact is available
- This way, we can specify which particular plays needs to be executed as show below

  `tags/catalogue.yaml`

  ```yaml
  - name: Install catalogue component
  hosts: catalogue
  become: yes
  tasks:
  - name: setup NPM source
    tags:
      - installation
    ansible.builtin.shell: "curl -sL https://rpm.nodesource.com/setup_lts.x | bash"

  - name: Install NodeJS
    tags:
      - installation
    ansible.builtin.yum:
      name: nodejs
      state: installed

  - name: download catalogue artifact
    tags:
      - deployment
    ansible.builtin.get_url:
      url: https://roboshop-builds.s3.amazonaws.com/catalogue.zip
      dest: /tmp
  ```

  `tags/inventory`

  ```inventory
  [catalogue]
  172.31.42.176
  ```

  `tags/ansible.cfg`

  ```cfg
  [defaults]
  remote_user = centos
  private_key_file = /home/centos/devops.pem
  inventory = inventory
  ```

- To execute this playbook, we run the command: `ansible-playbook catalogue.yaml -t deployment` and the tasks that has the tag `deployment` will only be executed.

## Inbuilt functions in Ansible

- Functions in Ansible are referred as filters in ansible

  `filters/demo.yaml`

  ```yaml
  - name: DEMO on filters
    hosts: localhost
    vars:
      website: https://www.joindevops.com/batch-74s
    tasks:
    - name: extract hostname
      debug:
        msg: "{{ website | urlsplit('hostname') }}"

    - name: set default value if not defined
      debug:
        msg: "{{ COURSE | default('DevOps') }}"
  ```

## Facts in Ansible

- Facts are the information that ansible collects at the time of execution
- Using these facts, we can identify in which distribution is ansible executing the commands
- This information is useful, we want to run the script in heterogeneous environment
- Even though, we never have different OS, OS version, Packages and its versions, services and their status, directory, permissions, etc. in DEV and PROD environments

  `heterogenous/facts.yaml`

  ```yaml
  - name: understand facts
    hosts: centos:ubuntu #all
    become: yes
    tasks:
    - name: print all the facts
      ansible.builtin.debug:
        msg: "All Facts: {{ansible_facts}}"

    - name: add user ubuntu
      ansible.builtin.command: adduser sivakumar
      when: ansible_facts['distribution'] == "Ubuntu"

    - name: add user centos
      ansible.builtin.command: useradd sivakumar
      when: ansible_facts['distribution'] == "CentOS"
  ```

  `heterogenous/inventory`

  ```inventory
  [centos]
  172.31.47.193 ansible_user=centos

  [ubuntu]
  172.31.46.88 ansible_user=ubuntu
  ```

  `heterogenous/ansible.cfg`

  ```cfg
  [defaults]
  remote_user = centos
  private_key_file = /home/centos/devops.pem
  inventory = inventory
  ```

- `ansible_user=centos` are said to be Host variables and using `hosts: centos:ubuntu`, we can specify more than one group of servers to select and run the playbook

## Dynamic Inventory

- This is necessary to fetch resources information from cloud providers such as AWS
- To work with AWS EC2 plugin on ansible, the files should have `aws_ec2.yaml` extension
- For example to fetch all the EC2 instances with the name web, we can use the following:

  `web.aws_ec2.yaml`

  ```yaml
  plugin: amazon.aws.aws_ec2
  regions:
    - us-east-1
  filters:
    tag:Name:
      - web
  ```

- To work with this, we should have `botocore` and `boto3` modules installed
- Then we can run: `ansible-inventory -i web.aws_ec2.yaml --list` to fetch the list of web hosts
- Then to run a command on all the hosts, we can run: `ansible aws_ec2 -i web.aws_ec2.yaml -e ansible_user=centos -e ansible_password=DevOps321 -m ping`
- We can use `%d` in vim to clear the file contents
