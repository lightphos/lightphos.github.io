---
layout: post
title: Vagrant HyperV
---


$ vagrant -v
Vagrant 2.1.5

$ vagrant box add centos/7 --provider hyperv
==> box: Loading metadata for box 'centos/7'
    box: URL: https://vagrantcloud.com/centos/7
==> box: Adding box 'centos/7' (v1804.02) for provider: hyperv
The box you're attempting to add already exists. Remove it before
adding it again or add it with the `--force` flag.

Name: centos/7
Provider: hyperv
Version: 1804.02

$ vagrant init centos/7

$ vagrant up --provider hyperv
Bringing machine 'default' up with 'hyperv' provider...
==> default: Verifying Hyper-V is enabled...
==> default: Verifying Hyper-V is accessible...
==> default: Importing a Hyper-V instance
    default: Creating and registering the VM...
    default: Successfully imported VM
    default: Please choose a switch to attach to your Hyper-V instance.
    default: If none of these are appropriate, please open the Hyper-V manager
    default: to create a new virtual switch.
    default:
    default: 1) ext-vSwitch
    default: 2) UbuntuSwitch
    default: 3) DockerNAT
    default: 4) Centos Virtual Switch
    default: 5) NATSwitch
    default: 6) Default Switch
    default:
    default: What switch would you like to use? 1
    default: Configuring the VM...
==> default: Starting the machine...
==> default: Waiting for the machine to report its IP address...
    default: Timeout: 120 seconds
    default: IP: 192.168.0.68
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 192.168.0.68:22
    default: SSH username: vagrant
    default: SSH auth method: private key
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!

