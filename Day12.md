# GitHub and Shell Scripting

Date: 16-08-2023

## Continuation with Github

- With SVN, we can select which files to track the changes made to it
- With SSH, we don't need username and password to authenticate ourselves with the remote server
- Create Public and Private Key pair on the client machine using `ssh-keygen -f <keypair-name>`
- Copy the Public key from the client machine to the server
- In the client machine, create a file called **config** inside `.ssh` directory and paste these contents inside that file

`~/.ssh/config`

```bash
Host github.com
  Hostname github.com
  User git
  IdentityFile ~/.ssh/github
```

- By default when accessing the github.com, the default username is git
- If we have multiple github accounts, then the config file looks as follow:

`~/.ssh/config`

```bash
# This is for learniguser account
Host github.com
  Hostname github.com
  User git
  IdentityFile ~/.ssh/github

# This is for cloudcampindia account
Host github.com-cloudcamp
  Hostname github.com
  User git
  IdentityFile ~/.ssh/cloudcamp
```

### How to push an existing folder to GitHub?

- Step 1: Initialise the folder as a Git repository `git init`
- Step 2: Add all the files to staging area `git add .`
  - `.` - Add all the files to the staging area
- Step 3: Commit all the changes to the files in the staging area: `git commit -m <message>`
- Step 4: Create a repo inside the Github server
- Step 5: Add remote URL as origin inside the git repo using: `git remote add origin <SSH-URL>`
  - We can see this configuration inside `.ssh/config` file
  - If we wanted to add another github URL to the origin, we use `-t` option
- Step 6: Push the files to the remote repo using: `git push origin master`
- To change the remote origin URL, we can use: `git remote set-url origin <URL>`
- To get the changes from the remote server synced in our local repo we use: `git pull`

## Shell Scripting

- Shell script files are saved with `.sh` extension
- The first line inside shell script is: `#!/bin/bash` which is called as shebang i.e. to inform linux kernel on how to run this script
- To print to the console: `echo` command
- We should never edit scripts inside the server, rather edit them in the editor and push them to the sever
- To execute a bash script, we use: `bash <filename>.sh`
- In programming, we use: DRY (Don't repeat yourself) principle

### Variables

- If a value is repeated multiple times inside a script, define a variable and declare that value to it
- And replace the value with that variable at each occurrence

```bash
PERSON1="Ramesh"
PERSON2="Suresh"
echo "Hello $PERSON1 and $PERSON2"
```

- Shell script at runtime fetches the actual values of the variables and executes the script
- Variables can be defined in multiple ways:
  1. Inside the script
  2. From command line

- Shell script executes the command inside `$()` and store its output in a variable as shown below

```bash
DATE=$(date)
echo "Today's date is $DATE"
```

- We can accept values to the variables using command line `bash 01-vars.sh Ram Raheem` and fetch them using `$1....$9`
- To get all the variables, we can use: `$@`
- To get the count of no: of variables passed: `$#`
- To get the name of the script: `$0`
- For e.g. Perform addition of 2 numbers that are provided at run-time

`01-vars.sh`

```bash
PERSON1=$1
PERSON2=$2
echo "Hello $PERSON1 and $PERSON2"

num1=$1
num2=$2
sum=$((num1 + num2))
echo "Sum of $num1 and $num2 is $sum"
```

- With the above script, we cannot perform string concatenation
- Let's say, we want to get the password information from the user inorder to connect to a database
- In this case, we can prompt the user to enter a password using:
- To hide something on the terminal when user is typing, we can use: `-s` option

```bash
echo "Please enter your username"
read USERNAME

echo "Please enter your password"
read -s PASSWORD

echo "Username: $USERNAME, Password: $PASSWORD"
```

### Data types in shell scripting

1. Number
2. Boolean
3. Arrays e.g. `PERSONS=("Ramesh" "Suresh" "Sachin")`

```bash
PERSONS=("Ramesh" "Suresh" "Sachin")
echo "First Person: ${PERSONS[0]}"
# Prints all persons
echo "Persons: ${PERSONS[@]}"
```
