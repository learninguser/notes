# Roboshop Implementation using Ansible Roles

Date: 04-09-2023

- In order to maintain DRY principle, we would like to take all the code that is common across modules and create functions from it
- In Ansible, functions can be implemented using Roles
- Ansible Roles consists of certain folder structure in which we can place the files such as `tasks/main.yaml` is responsible for executing the list of tasks that are in that role

  ```yaml
  - name: Installing Roboshop project
    hosts: "{{component}}"
    become: yes
    roles:
      - "{{component}}"
  ```

- All the tasks that are common across all the components are kept inside `common` role
- We can import that role using:

  ```yaml
  - name: Install NodeJS
    ansible.builtin.import_role:
      name: common
      tasks_from: nodejs
  ```

- When we want to replace a text with some other text, we can use placeholders using Jinja 2 syntax
- To place the content in the content-holder, we use templates in Ansible as shown below and use `ansible.builtin.template` module to serve this purpose

  `roles/web/templates/roboshop.conf.j2`

  ```conf
  proxy_http_version 1.1;
  location /images/ {
  expires 5s;
  root   /usr/share/nginx/html;
  try_files $uri /images/placeholder.jpg;
  }

  location /api/catalogue/ { proxy_pass http://{{CATALOGUE_HOST}}:8080/; }
  location /api/user/ { proxy_pass http://{{USER_HOST}}:8080/; }
  location /api/cart/ { proxy_pass http://{{CART_HOST}}:8080/; }
  location /api/shipping/ { proxy_pass http://{{SHIPPING_HOST}}:8080/; }
  location /api/payment/ { proxy_pass http://{{PAYMENT_HOST}}:8080/; }

  location /health {
  stub_status on;
  access_log off;
  }
  ```

  `roles/web/templates/roboshop.conf.j2`

  ```yaml
  - name: copy roboshop configuration
    ansible.builtin.template:
      src: roboshop.conf.j2
      dest: /etc/nginx/default.d/roboshop.conf
  ```

- The values are fetched from `vars/main.yaml` file and used at the time of playbook execution

  `roles/web/vars/main.yaml`

  ```yaml
  CATALOGUE_HOST: catalogue.joindevops.online
  USER_HOST: user.joindevops.online
  SHIPPING_HOST: shipping.joindevops.online
  CART_HOST: cart.joindevops.online
  PAYMENT_HOST: payment.joindevops.online
  ```

- Or we can also define `variables.yaml` in main directory where `main.yaml` file is present and pass it to the `main.yaml` as shown below using `vars_files`

  `variables.yaml`

  ```yaml
  - name: "install {{component}}"
    hosts: "{{component}}"
    vars_files:
      - variables.yaml
    become: yes
    roles:
      - "{{component}}"
  ```

## Handlers

- If we want to trigger a function when ever there is change in a file, we can use **handlers** as shown below

  `roles/web/handlers/main.yaml`

  ```yaml
  - name: restart nginx
    service:
      name: nginx
      state: restarted
      enabled: yes
  ```

  `roles/web/tasks/main.yaml`

  ```yaml
  - name: copy roboshop.conf
    ansible.builtin.template:
      src: roboshop.conf.j2
      dest: /etc/nginx/default.d/roboshop.conf
    notify:
      - restart nginx
  ```
