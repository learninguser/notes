# Ansible: Loops

Date: 30-08-2023

## Loops

- We use `item` as our iterable to access elements inside a loop

`ansible/loops.yaml`

```yaml
- name: loops example
  hosts: localhost # you no need to give user name and password through ansible command line
  tasks:
  - name: print the names
    ansible.builtin.debug:
      msg: "Hello {{item}}"
    loop:
    - Sivakumar
    - Raheem
    - John
```

## Roboshop project implmenetation

- Ansible Controller -> Ansible Server
