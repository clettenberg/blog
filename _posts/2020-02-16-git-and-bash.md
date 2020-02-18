---
layout: post
title: "Learn by Doing: Git and Bash"
date: 2020-02-16
author: Chase Clettenberg
author_login: clettenberg
author_email: clettenberg@gmail.com
author_url: http://cclettenberg.com
categories: tutorial
tags: tutorial git bash
---

#### What are we doing?

We are going to create a new git repo from an existing project

* [Bash Cheat Sheet](https://www.git-tower.com/blog/command-line-cheat-sheet/)
* [Learning Git](https://www.git-tower.com/learn/git/videos)

### Shell quick reference

```bash
# List files
ls

# Get current directory (present working directory)
pwd

# Change to root directory
cd

# Change to a specific directory
cd code/name_of_directory

# Go UP a directory
cd ..
```



## Add an existing project to GitHub
---

* Navigate to your project folder

  ```shell
  cd code/my_awesome_project
  ```

* Initialize a git repo

  ```shell
  git init
  ```

* Stage and commit your changes

  ```shell
  # Stage all changes
  git add -A

  # Commit changes
  git commit -m "Message goes here"
  ```

* Create a new Repository on GitHub
  * Follow this [GitHub guide](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-new-repository) and
create a repository with the same name as your directory `my_awesome_project`

* Add remote that points to your newly created repo and push changes
  ```shell
  # Add remote called 'origin'
  git remote add origin https://github.com/your-username/my_awesome_project.git

  # Push changes to origin
  git push -u origin master
  ```
* From now on, if you want to push changes to GitHub, simply

    ```shell
  # Stage all changes
  git add -A

  # Commit changes
  git commit -m "Message goes here"

  # Push changes to origin
  git push
  ```
