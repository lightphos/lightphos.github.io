---
layout: post
title: Xubuntu 17.10 Laptop LOT (List of things)
---

### General
#### Key-Gen (to connect to git lab)
#### Java 8 (from oracle)
```java version "1.8.0_172"
Java(TM) SE Runtime Environment (build 1.8.0_172-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.172-b11, mixed mode)
```

#### IntelliJ/Eclipse
For Java Devs esp.

#### Slack
    `https://slack.com/downloads/instructions/ubuntu`
    
#### curl 
`curl -sSL https://get.docker.com/ | sh`

#### Docker 
You need this to run your services in a container
```
curl -sSL https://get.docker.com/ | sh
sudo gpasswd -a $USER docker
newgrp docker
```
Note: this script expires on August 2018 for Ubuntu 17.10
 
#### AnyConnect
For VPN to your company network
https://www.vpnkeys.com/setup-cisco-anyconnect-ubuntu/

#### OpenConnect 
` sudo apt-get install network-manager-openconnect network-manager-openconnect-gnome `

Add the connection details

```
Menu -> Settings -> network connections -> add (+) -> Cisco AnyConnect Compatible VPN (openconnect)
```

![Screenshot_VPN-1](/content/images/2018/04/Screenshot_VPN-1.png)

![Screenshot_VPN2](/content/images/2018/04/Screenshot_VPN2.png)

Run from menu bar, getting need to upgrade client problem.

##### Command line 
Some companies only support connection from a windows m/c so:

sudo openconnect emeavpn-eudc.electrocomponents.com --os=win --user=<>

#### Skype
Use Pidgin
sudo apt-get install pidgin*
need to reboot

Then configure pidgin

![Pidgin-settings1](/content/images/2018/04/Pidgin-settings1.png)

![Pidgin-settings2](/content/images/2018/04/Pidgin-settings2.png)

UCCAPI/16.0.6001.1073 OC/16.0.6001.1073 (Skype for Business)


