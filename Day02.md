# Continuation of Linux Commands

Date: 01-08-2023

- AWS has its presence in several regions
- Therefore we should remember in which region are we creating our resources

- `grep <word-to-find> <filename>` -> To find a text inside a file
  - Case sensitive
  - To make it case insensitive: `-i` option
- `|` -> Pass the output of a command as an input to another command
  - E.g. `cat /etc/passwd | grep sbin`
- `head filename` -> First 10 lines of the file by default
  - `-n <no: of lines>` - print those no: of lines
- `tail filename` -> Last 10 lines of the file by default
- `wget <URL>` -> to download the file from internet
- `curl <URL>` -> Show the contents of the file onto terimnal
  - `-o` to write the output to a file
  - E.g. `curl -o /tmp/web.zip https://roboshop-builds.s3.amazonaws.com/web.zip`
- `unzip -o <filename>` - to decompress the compressed file
  - `-o` to overwrite the content
- `cut -d <delimiter> -f <index of the fragement>` -> Cut the string based on a delimiter
  - `-d` -> delimiter
  - `-f` -> fragment
  - The disadvantage with this command is that, we need to calcuate the fragment number manually
- `awk -F <delimiter> {print $<index of the fragement>F}`
  - To get the last fragment, we can use **NF** i.e `awk -F <delimiter> {print $NF}`
  - We can also use `awk` for column based filtering
- By default root userID is 0

## Editors

- vim editor: Visually improved editor
- `vim <filename>` - Creates a file and opens it if its not present. If its present, it opens the file directly
  - There are 3 modes in vim:
    1. Escape mode
        - `u` - To undo the changes made
        - `yy` - Yank/copy the line where the cursor is present + `p` for paste
          - `yy` + `10p` - copies the line and past it 10 times
        - `dd` - To delete one line
        - `gg` - To shift the cursor to the top of the file
        - `shift + g` - To shift the cursor to the bottom of the file
    2. Colon mode: command mode
        - `/<word-to-search>` - It will search for the word from the top of the file
          - `n` for searching the next occurence
        - `?<word-to-search>` - It will search for the word from the bottom of the file
        - `:q` - To quit the file
        - `:q!` - To force quit the file
        - `:noh` - No highlight i.e. it will unhighlight the previous search
        - `:se nu` - To display the line numbers in the file
        - `:se nonu` - To disable displaying the line numbers in the file
        - `:wq` - To save the changes and quit
        - `:s/<word-to-search>/<replace word>` - To search a word and replace it where the cursor is placed
          - `:2s/<word-to-search>/<replace word>` - To search a word and replace it in the 2nd line
          - `:%s/<word-to-search>/<replace word>/g` - To search a word and replace all the occurrences
    3. Insert mode

## Permissions in Linux

- User / owner -> The one who created the file
- group: The group to which user belongs to
- others: Other than user and group
- When a user is created in linux, a group with the same name as user is created by default
- `chmod <permissions> <filename>` -> change permissions to a file
  - For e.g. `chmod u+x <filename>` -> The user is given with executable permissions to execute the file
  - `chmod go+r passwd`
  - `chmod +x passwd`
  - `chmod -x passwd`
  - `chmod ugo+rwx passwd`
- Who can change the permissions of the file? owner or root user
- R - 4, W - 2, X - 1
  - E.g. `chmod 750 passwd`
- Usually to public and private keys, the premissions is never greater than 400
