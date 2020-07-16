---
template: blog-post
title: "Creation of SSH key for GitHub "
slug: ssh-key-github
date: 2020-06-28 19:02
description: Creation of SSH key for GitHub desktop
---
Github became one of the great software programmers love. After takeover by MicroSoft, it became even more go to option for free unlimited private repositories.

In this article, we will discuss about the creation SSH key for Github desktop application to commit the code from Gitbash window.

> Prerequisite is to have a github account and github desktop to be installed in system.

* Open Gitbash and run below command. 		 `git config --list`

Check for user.name and user.email properties in the output if already set. 

* If the properties are missing or not set , use below commands to set.

```
git config --global user.name "<name>"
git config --global user.email "<email>"
```

* Execute  again config --list command and verify whether name and email set
* Run below command to create SSH key.

```
ssh-keygen -t rsa -b 4096 -C "email id"
```

Enter and give a passphrase to be used as a password in ssh auth.

```
eval $(ssh-agent -s)
```

This is evaluate the created ssh file.

```
ssh-add ~/.ssh/id_rsa
```

It will add ssh to id_rsa. It will ask for passphase which you created in previous step.

Ola.... its done... But you have to link the created machine specific SSH to github which opens the gates to the requests coming from your machine to Github servers.

By using below command copy the ssh key of rsa pub file content to clipboard.

```
clip < ~/.ssh/id_rsa.pub
```

Open your github settings and navigate to SSH keys option and add key to it.

That's it. Now you can seamlessly commit the changes to github from your machine.

#### **Some of the useful git commands:**

| Description        | Command |
| ------------- |-------------|
|To take master checkout	      |git checkout master |

