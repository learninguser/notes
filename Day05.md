# Continuation of Linux Commands

Date: 04-08-2023

- `find <where-to-find> -name "hello"`: Find a file with name "hello" inside all the directories present inside linux filesystem

## Linux filesystem

- / - Linux start Directory
- /boot - At the time of boot-up, linux searches for this directory and a file called **grub.cfg** inside grub2 directory
- /root - It is home folder of super admin user
- /dev (devices) - Information realted to harddisks, external devices, keyboards etc
  - `tty` - Keyboard
- /etc (Extra Configuration) - Configuration of all the packages and services
- /home - All the other users directories are present here
- /lib or /lib64 - All the dependencies of the packages and commands
  - It is similar to dlls in windows
- /media - cd, dvd, will be mounted here
- /mnt - It is used to mount the external harddisk or Network File Storage (NFS)
- /opt - optional, third party softwares like tomcat
  - Usually it is recommended to store all those packages we want to download and run manually
- /proc - All the information related to the running process is stored here
  - Once the system is stopped, everything present in this directory are deleted
  - E.g. `cat /proc/meminfo`, `cat /proc/cpuinfo`
- /run - At the time of OS loading, it will need some temporary filesystem. It will use this directory for that purpose
  - Once the server is stopped, all th contents present in this directory are also deleted
- /bin - All the binaries for executing the commands such as pwd, ls, vim, top etc are stored here
- /sbin - System binaries i.e. Admin related commands
- /temp - For storing temporary data. The data present in this directory will be deleted once the server is restarted
- /var (Variables) - All the log files, system related messages etc.
  - To scroll through the messages, we can use `less` command
    - `shift + g` to scroll to the end
    - `gg` to scroll back to up

## inode, symlink/softlink, hard link, Tar command

- inode: For everything in the linux filesystem, there is a number associated to it i.e. a pointer to where it is stored in the harddisk.
  - We can fetch this information using: `ls -ltri`
- Softlink / Symbolic link: It is the shortcut that points to the original file which points to the inode value
  - To create a symlink, we use: `ln -s <source-file> <link-name>`
  - The inode value of the symlink file is **different** than the original one
  - If the source file is deleted, the softlink throws No such file or directory exception
- Hard link: Creates a copy of the file but points to the same inode value
  - To create a hard link, we use: `ln <source-file> <link-name>`
  - To find the source file of the hardlink, we can use the inode number
    - `find <where-to-search> -inum <inode-number>`
  - If the source file is deleted, the hard link still holds the data
- To extract the .tar.gz file, we use: `tar -xf <.tar.gz file path>`
  - `-x` - Extraction
  - `-f` - filename

## Crontab

- Scenario: We need to monitor a linux server all the time i.e. its memory, cpu resource utilisation
- For this, we create a shell script and schedule it to run every hour
- To open crontab editor: `crontab -e`
- Syntax: `* * * * * <command to execute>`
  - Ref: [https://crontab.guru/](https://crontab.guru/)
  - In this syntax, we want to execute a command **At every minute**
