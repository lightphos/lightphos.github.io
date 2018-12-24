---
layout: post
title: Running Docker when on a Cisco AnyConnect VPN
date: '2018-04-21 22:15:02'
categories: 'Tech'
author: Satish Ramjee
tags:
- docker
- vpn
- windows
- cisco-anyconnect
- open-connect
---

If you are working with Cisco AnyConnect you will find that the normal docker address of 192.168.99.100 will be blocked. This means that although you can start the docker server (docker-machine start), you won't be able to connect to it with the client (docker ps -a, for example), even when you've set the docker variables (docker-machine env).

## Work around

Change the docker server default IP address and use the open source open connect version

## Change the Docker server default address

Edit C:\Users\<username>\.docker\machine\machines\default\config.json and changing "HostOnlyCIDR": "10.168.99.1/24".  

You might optionally need to created a new HostOnly network adapter for the "10.168.99.1/24" in the VirtualBox default machine. You can do this by copying and pasting the existing one, and changing 192.* to 10.*. 

Try without first and see if it works. 

Any running docker images (not the docker-machine itself) might need restarting after connecting to the VPN.

## Use Open Connect
Download the windows version:

[https://github.com/openconnect/openconnect-gui/releases/download/v1.5.3/openconnect-gui-1.5.3-win32.exe
](https://github.com/openconnect/openconnect-gui/releases/download/v1.5.3/openconnect-gui-1.5.3-win32.exe)

Start the GUI, add a new Profile, and enter your company VPN hostname for the hostname (no need  to enter any protocol or ports), and enter your VPN username.  

`NOTE: Make sure the Batch mode is not selected otherwise it will try and connect multiple times without prompting (as it will assume the credentials are in a default location). This could block you out from your company VPN, needing them to reset the connection`

Click "Connect", and then enter your VPN password prefix + your Symantec 6 digit code from your app in the prompt and it should connect.



