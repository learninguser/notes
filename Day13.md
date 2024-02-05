# Shell Scripting

Date: 17-08-2023

## Continuation with Shell Scripting

### Conditions

- Ex 1: Given a number in realtime check if its greater than 10 or not

```bash
NUMBER=$1
if [ $NUMBER -gt 10 ]
then
  echo "Number is greater than 10"
else
  echo "Number is less than 10"
fi
```

- Ex 2: Check if MySQL is installed or not
- Steps to solve this:
  1. Check if the user who is executing the script is a root user or not
      - This we can get using `id -u` command
      - For a root user, the result would be 0
      - For non-root user, it will be non-zero
      - If its non-zero, we should stop executing further statements in the script using: `exit 1`
  2. If its a success, we install MySQL using: `yum install mysql -y`

- **Note: Shell script doesn't stop if it encounters an error by default, therefore its the developers responsibility to handle it properly**
- We can get the status of the execution of a statement using: `$?`

```bash
USERID=$(id -u)
if [ $USERID -ne 0 ]
then
  echo "Error :: Please run the script with root privileges"
  exit 1 # 1 - 127 indicates its a failure
fi

yum install mysql -y

if [ $? -ne 0 ]
then
  echo "Installation of MySQL package is unsuccessful"
else
  echo "Installation of MySQL package is successful"
fi

yum install postfix -y

if [ $? -ne 0 ]
then
  echo "Installation of postfix package is unsuccessful"
else
  echo "Installation of postfix package is successful"
fi
```

### Functions

- Using Functions, we can keep some lines of code inside a block and call it when ever we need it
- Syntax:

```bash
FUNCTION_NAME(){

}
FUNCTION_NAME
```

```bash
check_status(){
  if [ $1 -ne 0 ]
  then
    echo "$2 ... FAILURE"
  else
    echo "$2 ... SUCCESS"
  fi
}

USERID=$(id -u)
if [ $USERID -ne 0 ]
then
  echo "Error :: Please run the script with root privileges"
  exit 1 # 1 - 127 indicates its a failure
fi

yum install mysql -y
check_status $? "Installing MySQL"

yum install postfix -y
check_status $? "Installing postfix"
```

- When scripting, its always import to maintain persistent logs i.e. will remain on harddisk until we delete it to debug the issues instead of printing it to console

#### Redirections

- `ls -l 1> ls.log` or `ls -l > ls.log` - To store the output of the command `ls -l` inside ls.log file
  - `1>` - Only redirects outputs with exit status 0 i.e. success
  - `2>` - Only redirects outputs with exit status other than 0 i.e. Failure
  - `>` - Replaces the previous content with the current content
  - `&>>` - Redirect both success and failure outputs
- `date +%F` - Returns the date in YYYY-MM-DD format
- `date +%F-%H-%M-%S` - Returns the date and time in YYYY-MM-DD-HH-MM-SS format

```bash
DATE=$(date +%F)
SCRIPT_NAME=$0
LOG_FILE=/tmp/$SCRIPT_NAME-$DATE.log

check_status(){
  if [ $1 -ne 0 ]
  then
    echo "$2 ... FAILURE"
  else
    echo "$2 ... SUCCESS"
  fi
}

USERID=$(id -u)
if [ $USERID -ne 0 ]
then
  echo "Error :: Please run the script with root privileges"
  exit 1 # 1 - 127 indicates its a failure
fi

yum install mysql -y &> LOG_FILE
check_status $? "Installing MySQL"

yum install postfix -y &> LOG_FILE
check_status $? "Installing postfix"
```

#### Colors in shell scripting

- For Red: `echo -e "\e[31m Hello World"`
  - `-e` - Enabling colors
- For Green: `echo -e "\e[32m Hello World"`
- For White: `echo -e "\e[0m Hello World"`

```bash
DATE=$(date +%F)
SCRIPT_NAME=$0
LOG_FILE=/tmp/$SCRIPT_NAME-$DATE.log

# COLORS
R="\e[31m"
G="\e[32m"
W="\e[0m"

check_status(){
  if [ $1 -ne 0 ]
  then
    echo -e "$2 ... $R FAILURE $W"
  else
    echo -e "$2 ... $G SUCCESS $W"
  fi
}

USERID=$(id -u)
if [ $USERID -ne 0 ]
then
  echo "Error :: Please run the script with root privileges"
  exit 1 # 1 - 127 indicates its a failure
fi

yum install mysql -y &> LOG_FILE
check_status $? "Installing MySQL"

yum install postfix -y &> LOG_FILE
check_status $? "Installing postfix"
```

### Loops

- Syntax:

  ```bash
  for i in {1..100}
  do
    echo $i
  done
  ```

- Installing list of packages at runtime:

  ```bash
  for i in $@
  do
    yum install i -y
  done
  ```

- How do we decide which instance type to choose?
  - Ans: Based on the benchmark tests such as how much load the instance can handle etc
  - We can use some tools such as Apache Benchmark testing tool which sends 10K requests at one time
- To perform reverse search, we can use: `ctrl + r`
