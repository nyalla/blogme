---
template: blog-post
title: Automating manual tasks via Winscp scripts
slug: winscp-scripts
date: 2022-04-08 14:09
description: "Automating manual tasks via Winscp scripts deployment "
---
I have been always working on the never ending tasks of moving jars to and fro from remote systems. From initial days I am using winscp as FTP tool to download and upload files. Sometimes its hectic I have to move the file to multiple servers. The task was frustrating and have to do it manually.

Then I was reading an article where they mentioned about Winscp scripts/commands that we can use it from local command line.  Seeing that like I felt like Why I never felt to search about this. I should divorce the feeling of "Its working , don’t touch". 

So asusual we will go with a usecase and lets see how we can do this with scripts.

Use case: Have to deploy jar files into remote Linux system and deploy it in the server and restart the server

Seeing the usecase, it looks like manual effort in much sense. Lets break down it into further steps. 	

1. Copying local files to remote location
2. Moving from tmp location to deployment folder
3. Run the commands to deploy and restart the server

Prerequisites: You need installed version of Winscp in local machine
Command line from the winscp installed location

**Automation script**:

```


open sftp://username:password@@IPADDRESS 

cd /tmp/deployFolderTemp

\#remove existing files 

rm *

put C:\local

\#copy files to  deploy folder 

call sudo -u username cp cd /tmp/deployFolderTemp/* /server/standalone/deployments/jars/

\#deploy  cd /server/standalone/deployments
call sudo -u username <command to restart>

exit
```

We can replicate this to many servers and reuse the same. I know this is a small task , but in Agile environment frequent deployments need to be automated where deployment pipeline is not placed.



Further we can enhance this with Powershell scripts, so that we can do programatic decisions before and after deployment processes.

Happy coding