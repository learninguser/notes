# Continuation of Linux Commands

Date: 02-08-2023

## User Management

- Before we perform any operations related to user maangement, we should be a root user which can be switched using `sudo su -`
- To check if we are a root user or not, we can identify it on the terminal with `#` symbol or with `id` command
- For root user, the uid is 0
- Creating User: `useradd <username>`
  - Linux starts creating new users with uid >= 1000
  - Below this Linux reserves them for system users
- Setting password to user
- Reading the user information: `id <username>`
- Updating the user information
- All the information related to the users is present inside `/etc/passwd` file
  - Username, uid, gid, home location, which shell he has access to
  - We can also access the same information using: `getent passwd`
- setting a password for a user: `passwd <username>`
- By default Linux disables the password based authentication
- To enable it, we need to modify the content of `/etc/sshd/sshd_config` file (in the line where it says PasswordAuthentication No)
  - sshd_config is a crucial file, any errors in the file, removes the whole access to the server
  - Therefore, we store a backup of this file
- **Before restarting the service, we should test if the configuration that is present inside the sshd_config file is correct or not**
- To test the configuration, we can use `sshd -t`
  - If there're any errors, it will print on to the terminal
- If there are no errors, we should restart the service using `systemctl restart sshd`
- Every user has 2 types of groups:
  1. Primary Group (gid)
  2. Secondary Group (groups)
- To create a group: `groupadd <groupname>`
- To add a user to a group: `usermod -g <groupname> <username>`
  - `-g`: primary group
  - `-G`: secondary group
- To list all groups in the system: `getent group`
- To add one more secondary group to a user: `usermod -aG <groupname> <username>`
  - `-a` -> appending
- To delete a user from a group: `gpasswd -d <username> <groupname>`
- An user should always be a part of one group
- To delete a user completely:
  1. Assign the group of the user to the same as his username: `usermod -g <groupname> <username>`
  2. Delete the user using: `userdel <username>`
- To delete a group:
  1. There shouldn't be any users part of that group, therefore we should remove all the users that are part of that group
  2. `groupdel <groupname>`

## Enable SSH access to a user

- User should create a public and private key pair at first
- Share the public key with the administrator
- Admin creates a `.ssh` folder inside his home directory
- Inside the .ssh directory, we should create file called `authorised_keys`
- Change the permissions to the file as: `chmod 400 authorised_keys`
- Change ownership to the folder: `chown <username>:<groupname> -R <foldername>`

## Process management

- Everything in Linux is a process
- We can see the list of all process that are running the system using: `ps -ef`
  - To see the list of process that are active in the current terminal: `ps`
- Each process has an ID associated with it and it is called as Process Instance ID
- Root user gets the PID of 0
- In addition to that it also has a Parent Process ID (PPID) linked to it
- There are 2 kinds of process:
  1. Foreground process
  2. Background process: `&`
- To kill a process: `kill <process-id>`
  - To kill a process forcefully: `kill -9 <process-id>`

## Package management

- To install package:
  - on centos: `yum install <package-name> -y`
  - on ubuntu: `apt install <package-name> -y`
  - on Amazon Linux:
    - `sudo amazon-linux-extras install epel -y`
    - `sudo yum install <package-name> -y`
- To remove a package: `yum remove git -y`
- To get the list of all packages (including installed): `yum list all | wc -l`
  - `wc -l` -> counts number of lines in the previous command
- To get the list of all installed packages: `yum list installed | wc -l`
