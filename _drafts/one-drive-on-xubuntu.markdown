---
layout: post
title: Setup Ubuntu to work for Git/Gitlab etc
---

Install git 

`sudo apt install git`


Add SSH Key

`ssh-keygen -t rsa`

return all options

File will be in .ssh directory

Add your key (cat .ssh/id_rsa.pub) into gitlab like so:

![gitlab-ssh-keys](/content/images/2018/04/gitlab-ssh-keys.png)

To grab your code

`git clone git@.....`

All done
