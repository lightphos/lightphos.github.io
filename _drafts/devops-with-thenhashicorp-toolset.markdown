---
layout: post
title: DevOps with the HashiCorp Toolset and GCP
---

Small Self Consistent Services need to be able to be developed independently and deployed independantly...but

Before services [A, B] occupied the same memory, now A and B are discrete and separated out. 

A depends on B where B may no longer be in the same machine, and now communication is slower (ms rather than ns latency).

Discovery - where is something I depend on?

Normally we put load balancers (LB) in front of each service and use hardcoded IP addresses of the LB. This increases costs. It is manually managed hence time and resource intensive. LB itself is SPoF. Network latency is also doubled.

Solution is to use a central (redundant) registry. A now talks to the registry to find B.
    
Configuration - how to avoid having to manually configure each instance
Segmentation - what do we want to talk to what, levels of segragation
Security - securing who talks to whom, using certs

Service Mesh is Discovery, Config, Segmentation, Security, Telemetry, Monitoring (see Lyft -> Envoy).

#### Hashicorp toolset

Consul:
https://www.youtube.com/watch?v=mxeMdl0KvBI

Nomad:
https://www.youtube.com/watch?v=s_Fm9UtL4YU
Devs:
    - write Jobs. Version of image, how many instances, roll out strategy
Operations: 
    - restarting images, farm based approach, automatically deploy to working servers
    - improve hardware utilisation (normally <2%) increase to 20% effectively replace 10 machines with 1
    
Nomad acts as a 
    - container platform, windows/legacy apps. 
    - work schedular
    - high performance computing - C1M - how quick to spin up 1M containers
    
Terraform:
    - Infrastructure as code
    - Provisioning
    - Regularly and evolve
    - Destroy
    
Application Delivery with Hashicorp tools:
https://www.youtube.com/watch?v=wyRtz_tdJes


Service Autodiscovery with nomad consul
https://www.youtube.com/watch?v=Qf3w02F5wfw



### GCP with Centos 7    

Using google cloud compute and centos 7 instance

Project -> Compute Engine -> VM Instances.

(assume all provisioned and ssh keys added)

Get the external ip eg (35.xxx.6.255)

ssh -i ~/.ssh.id_rsa username@35...


Install docker
https://linuxize.com/post/how-to-install-and-use-docker-on-centos-7/

Once installed
```
sudo systemctl start docker
[satishramjee@instance-1 ~]$ sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[satishramjee@instance-1 ~]$ sudo docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
nginx                 latest              06144b287844        3 months ago        109MB
vertx/vertx3          latest              ea0f0728aafc        5 months ago        557MB
hashicorp/http-echo   latest              a6838e9a6ff6        19 months ago       3.97MB
```

### NOMAD and CONSUL
Install nomad (note: start nomad first then consul)

nomad agent -dev -consul-address 1.2.3.4:8500 -config nomad.conf

NOTE THIS IS JUST FOR DEMO - YOU WOULD NEED TO PROTECT ACCESS TO YOUR SERVER. One way is via ssh tunnelling if you only wish to test.

nomad.conf
```
addresses {
  http = "10.12x.0.x"
}
```


Install consul
wget https://releases.hashicorp.com/consul/1.2.3/consul_1.2.3_linux_amd64.zip
unzip ..
mv consul /usr/local/bin/

consul agent -dev -ui -client {internalip} (or it won't expose to outside world)

Ok now to run a nomad image example (redis)


export NOMAD_ADDR=http://10.12x.0.x:4646
[sramjee@instance-1 nomad-work]$ nomad job run example.nomad
```
 job "nginx" {
    region = "office"
    datacenters = ["west"]
    type = "service"
    all_at_once = false
    group "nginx" {
        count = 1
        restart {
            attempts = 10
            delay = "15s"
            interval = "5m"
            mode = "delay"
        }

        # This is checking the nginx service
	    task "nginx-ck" {
	     driver = "exec"
           config {
               command = "tail"
               args = ["-f", "/dev/null"]
           }
           resources {
                   network {
                       mbits = 1
                       port "nginx" {}
                   }
           }
            service {
              name = "nginx"
              tags = ["frontend", "dev"]
              port = "nginx"
              check {
                type = "tcp"
                interval = "5s"
                timeout = "2s"
              }
            }
	    }
        task "nginx" {
            driver = "docker"
            logs {
                max_files = 3
                max_file_size = 100
            }
            config {
                image = "nginx"
                # we want the world to see this
                network_mode="host"
                ## this is the port number inside the container
                port_map {
                    service_port = 80
                }
            }

            resources {
                cpu = 512
                memory = 2048
                network {
                  mbits = 82
                  port "service_port" { 
                     # fix the host port to 80 (outside the container)
                     static = 80
                  }
                }
                disk = 2048
            }

            env {

            }
        }
    }

}

```

### DNS with BIND-UTILS

yum install bind-utils

```
dig @10.128.0.2 -p 8600 nginx.service.consul

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7_5.1 <<>> @10.128.0.2 -p 8600 nginx.service.consul
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16402
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;nginx.service.consul.          IN      A

;; ANSWER SECTION:
nginx.service.consul.   0       IN      A       127.0.0.1

;; ADDITIONAL SECTION:
nginx.service.consul.   0       IN      TXT     "consul-network-segment="

;; Query time: 0 msec
;; SERVER: 10.128.0.2#8600(10.128.0.2)
;; WHEN: Fri Sep 14 22:29:20 UTC 2018
;; MSG SIZE  rcvd: 101


```


### Terraform

Terraform/Terragrunt

https://github.com/gruntwork-io/terragrunt

https://releases.hashicorp.com/terraform/0.11.8/terraform_0.11.8_linux_amd64.zip

Example nomad provider code: 
https://github.com/hashicorp/terraform-container-deploy-nomad

It's an echo:
```
 image = "hashicorp/http-echo:${version}"
```
Edit it to point to nomad server

To run:

terraform init
terraform plan
terraform apply -var “version=latest”

Using terragrunt to help with commonality
https://github.com/gruntwork-io/terragrunt#use-cases



Hook into Apache?
https://github.com/hashicorp/consul-template/blob/master/examples/apache.md


IaaS
AWS, Azure, GCP, Bluemix, OpenStack, VMWare

PaaS
Heroku, K8Sm Lambda

SaaS
Datadog, Gitlab, Frog, 

