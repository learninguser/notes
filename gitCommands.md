# Further notes on Git Commands

- `git commit -am "message"` - Adds all files to staging area and commits with a message
- `git log -n 2` displays the details of first 2 commits
- `git log --author="Pavan Kumar Akula"` displays all the commits made by the Author "Pavan Kumar Akula"
- `git log --since=2024-02-15` displays all the commits made since 15th Feb 2024
- `git log --since="1 week ago"` display all the commits made in the past 1 week
- Commit Hashes is a checksum generated with data and is a 40 character long string generated using SHA1 algorithm
- A snapshot is generated using SHA of the current commit + SHA of previous commit (aka parent commit) + data + metadata
- **HEAD** always points on to the **tip of the current branch** where ever we're working
- `git diff` - shows the changes that were made compared to previous commit
  - `git diff --staged` shows the changes in the file that is in the staged area compared to previous commit
- Once a commited file is deleted manually its actually moved to the recycle and perform `git commit`
- But a much better way would be to use: `git rm <filename>` and perform `git commit`
- Lets consider a case where we made a mistake with the local version of the file, we would like to pull the version of the file from the remote repo to local repo, we can use: `git checkout -- <filename>`. In this case, the file is in unstaged area
- If the file is in the staged area, then we use: `git reset HEAD <filename>` to bring it back into the unstaged area
- Once a commit is made, we cannot change it except if its the last commit in the repo
  - In this case, we can use: `git commit --ammend -m "<messsage>"`
- **RESET means you're about to loose data**
- If we want to checkout to a certain commit where the application is working perfectly fine, there are couple of ways to do it
  - Method 1: `git checkout <commit-id> -- <filename>` and then `git reset HEAD <filename>`
  - Method 2:

## Git Reset

- There are three kinds:

1. Soft: Makes no change to the files rather moves the HEAD pointer to the specifed commit ID and starts recording from thereon
    - `git reset --soft <commit-id>`
2. Mixed: This is the default case
3. Hard: Must be used with **CAUTION** as it can do damage to the repo i.e. deletes the files
    - Make the local repo identical to remote repo

- Shortcut: `git reset HEAD~1` to revert to previous commit and `git reset HEAD~2` to revert 2 commits before the current commit
- To ignore all `.js` files except one file, we can use for e.g.: `!app.js` in the `.gitignore` to track changes made to app.js file
- To remove a tracked file from the commit history: `git rm --cached <filename>`
- Similar to `ls -la`, we have `git ls-tree HEAD` to show the directory structure at that commit
  - `tree` -> represents a directory
  - `blob` -> represents a file
- We can refer the same using: `HEAD~1` for the parent, `HEAD~2` for grand parent etc
- `git log --oneline -3` prints last 3 commits in the commit history
- `git log --grep "data"` prints all the commits that contains the word "data" in its commit message
- To print all the commits between a certain range i.e. SHA1 and SHA2, we use: `git log <SHA1>..<SHA2>` where SHA1 is the start commit Hash
- `git log --format=oneline` list down all the commits with its complete SHA value and the commit message

## Branching

- `git branch` - To list all the branches in the current repository
- `git branch volumes` - creates a branch named volumes
- `git checkout volumes` - Switches from master branch to volumes branch
- `git checkout -b network` - Creates a new branch from volumes named network and switches to it
- `git checkout -- abc.txt` - Removes all the local changes made to the file and reverts back to the previous commit
- `git branch -m volume volumes` - Renames the branch from volume to volumes
- `git branch -d volumes` - Removes the volumes branch i.e. there shouldn't be any extra commits inside it
  - `git branch -D volumes` - In this case, irrespective of the changes made, delete the branch
- To view the commits of all branches at one place: `git log --oneline --graph --decorate --all`
- **stash** is a reserved area where we can put things for meanwhile and we can bring them back
