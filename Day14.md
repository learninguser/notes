# Shell Scripting

Date: 18-08-2023

- Task 1: Install multiple packages through command line
- Steps to solve this task:
  1. User should have root access
  2. While installing store the logs
  3. Implement colors for user experience
  4. Before installing it is always good to check whether package is already installed or not
      - If installed skip, otherwise proceed for installation
  5. Check successfully installed or not

  ```bash
  #!/bin/bash

  DATE=$(date +%F)
  LOGSDIR=/home/centos/shellscript-logs
  # Logfile name format: /home/centos/shellscript-logs/script-name-date.log
  SCRIPT_NAME=$0
  LOGFILE=$LOGSDIR/$0-$DATE.log
  USERID=$(id -u)
  R="\e[31m"
  G="\e[32m"
  N="\e[0m"
  Y="\e[33m"

  if [ $USERID -ne 0 ];
  then
      echo -e "$R ERROR:: Please run this script with root access $N"
      exit 1
  fi

  check_status(){
      if [ $1 -ne 0 ];
      then
          echo -e "Installing $2 ... $R FAILURE $N"
          exit 1
      else
          echo -e "Installing $2 ... $G SUCCESS $N"
      fi
  }

  # all args are in $@
  for i in $@
  do
      yum list installed $i &>>$LOGFILE
      if [ $? -ne 0 ]
      then
          echo "$i is not installed, let's install it"
          yum install $i -y &>>$LOGFILE
          check_status $? "$i"
      else
          echo -e "$Y $i is already installed $N"
      fi
  done
  ```

- Task 2: Delete Log files older than 2 weeks inside the following Logs directory: /home/centos/app-logs
- Note: only .log files should be deleted, dont delete any other files
- Steps to solve this task:
  Step 1: go to the folder,  get all the log files with extension of .log
  Step 2: check the file modification date
  Step 3: If modification date is more than 2 weeks old, delete the files
- To create files with older date: `touch -d 20230801 cart-2023-08-01.log`, using this a file with the name cart-2023-08-01.log is created with the modification date as Aug 1st 2023
- To find files that are 2 weeks older: `find /var/log -name "*.log" -type f -mtime +14`
  - `+14` indicates 14 days

  ```bash
  #!/bin/bash

  APP_LOGS_DIR=/home/centos/app-logs

  DATE=$(date +%F-%H-%M-%S)
  LOGSDIR=/home/centos/shellscript-logs
  # /home/centos/shellscript-logs/script-name-date.log
  SCRIPT_NAME=$0
  LOGFILE=$LOGSDIR/$SCRIPT_NAME-$DATE.log

  FILES_TO_DELETE=$(find $APP_LOGS_DIR -name "*.log" -type f -mtime +14)

  echo "script started executing at $DATE" &>>$LOGFILE
  while read line
  do
      echo "Deleting $line" &>>$LOGFILE
      rm -rf $line
  done <<< $FILES_TO_DELETE
  ```

- To read line by line from a variable, we use `<<<`
- A shell script can be executed in multiple ways:
  - `bash <script-name>` --> for this no need of execute permission
  - `./<script-name>` --> it should have execute permission which can be given using: `chmod +x <script-name>`
- To schedule a script to run, we use `crontab` in edit mode: `crontab -e`
  - `* * * * * <absolute-file-path>`
  - Note: **In crontab, we should only specify absolute filepath**
- We can also view the crontab logs inside `/var/log/cron` file

## Configuring Linux server for sending emails

- In order to send an email from Linux Server, we need to configure it using a 3rd party tool such as postfix
- Follow the steps that is present in this [documentation](https://github.com/sivadevopsdaws74s/concepts/blob/master/gmail.MD) to configure the same on CentOS 8 server
- Kernel packages are updated only when we perform upgrading the OS
