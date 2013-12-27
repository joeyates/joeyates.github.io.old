--- 
layout: post
typo_id: 14
title: Shared Git Repositories
---
When you want to have a central shared git repository to push commits to, there are two common configurations:
<ol>
	<li>Everybody uses the same user for commits - easy to set up, but very messy,</li>
	<li>Create a system user for each committer, create a system group for each project and add users to relevant groups.</li>
</ol>

<!--more-->

Below is an explanation of how to set up and manage configuration 2.

# Create User 'git'

Create a 'user 'git' to manage projects.

    $ sudo useradd --shell /bin/bash --home-dir /var/git --create-home git

# A Directory for Repositories

    $ sudo su - git
    $ mkdir /var/git/repositories

# Create a Project

    $ cd /var/git/repositories
    $ mkdir [PROJECT_NAME]
    $ cd [PROJECT_NAME]
    $ git --bare init

Create a group for the project:

    $ sudo groupadd [PROJECT_NAME]

Add relevant users to the group:

    $ sudo usermod --append --groups [GROUP_NAME] [USER_NAME]

Give the group write permissions to the repository:

    $ sudo chgrp -R [GROUP_NAME] /var/git/repositories/[REPOSITORY_PATH]
    $ sudo chmod -R ug+w /var/git/repositories/[REPOSITORY_PATH]
    $ chmod -R o-w /var/git/repositories/[REPOSITORY_PATH]

# Have Git Handle Group Permissions

In .git/config:

    [core]
      sharedrepository = 1

#	Use

Now, when you use the remote shared repository, you can use your personal account:

    $ git clone ssh://my-user@example.com/path/to/repo
