# GIT and Github, Shell Scripting

Date: 15-08-2023

## Disadvantages with manual setup of the project

- So far all the steps we have executed manually
- Due to this, the whole process is time taking
- Easy to do mistakes
- Takes more time if we want to repeat the same process multiple times

## Shell Scripting

- Keep all the commands inside a script and execute the script inside the server
- Shell is native scripting in Linux
- To do some automations with:
  - internal system: prefer Shell scripting
  - external system: prefer Python scripting
- There are 2 types of coding:
  1. Programming: Development i.e. less time, more performance, less resources etc using DSA, Design Patterns etc
  2. Scripting: Automation insted of Manual tasks
- With in coding, we have:
  1. Variables
  2. Data Types
  3. Conditions
  4. Loops
  5. Functions

## GIT and Github

- When writing scripts, we should never save them in our Local PC rather store them in repos
- This ensures saftey and security for the storage of the code
- In addition to that, we can maintains different Versions of the scripts
- This allows us to revert to the previous versions when ever we want
- We can have different feature branches where we develop features and merge them into master by raising a pull request
- On the master branch, we have the production code
- In addition to that, we can also track which developer has done what
- Popular tool: GIT
- Popular platforms that hosts this GIT are:
  - Github
  - Gitlab
  - Bitbucket
- Each platform offers, Public and Private installations
- In Git, We can store everything inside a repository and push it to Github server
- Gitbash tool offers SSH service and works as a client to connect to any Git servers
- Github is a de-centralised distribution system i.e. everything is not in a single place rather its distributed
- Workspace -> Staging (temporary) area -> local repo -> central repo
