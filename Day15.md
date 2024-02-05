# Shell Scripting

Date: 21-08-2023

- Task 3: write a shell script that should check disk usage every one hour, and send alert email if it is consuming more than 80%
- Steps:
  1. check disk memory
  2. compare with threshold (10%)
  3. if more than threshold trigger alert email
- `df -hT` --> to check disk memory
- `df -hT | grep -v tmpfs` - Returns the entries that doesn't have tmpfs in it
- To apply OR condition: `df -hT | grep -vE 'tmpfs|FileSystem'`
- When mounting an extra partition to the instance, we should ensure that its in the same AZ as our instance to which it needs to be attached

## SED Editor

- Streamline Editor i.e. running version of the editor
- By default: SED makes temporary changes to the file
- To create a line: `sed -e '1 a Good morning' <filename>`
  - `a` - append
  - `1` - after 1st line
- To insert a text before line 1: `sed -e '1 i Good morning' <filename>`
  - `i` - insert
- To make permanent changes, we use `-i` option
  - `sed -i '1 i Good morning' <filename>`
- To search a word and replace it with another word: `sed -e 's/<word-to-find>/<word-to-replace>/' <file-name`
  - `s` - Substitute
  - Note: This will only replace the first occurence in every line
- To replace all the occurences, we need to specify `g`, g for global
  - `sed -e 's/<word-to-find>/<word-to-replace>/g' <file-name`
- To delete lines that contains a word: `sed -e '/<word-to-delete>/ d' <file-name`

- Implementation scripts:
  - [disk_usage.sh](https://github.com/sivadevopsdaws74s/shell-script/blob/master/disk_usage.sh)
  - [mail.sh](https://github.com/sivadevopsdaws74s/shell-script/blob/master/mail.sh)
  - [template.html](https://github.com/sivadevopsdaws74s/shell-script/blob/master/template.html)

## Roboshop project Implementation: Mongodb

```bash
git clone https://github.com/sivadevopsdaws74s/roboshop-shell.git
git checkout e0eed71
```
