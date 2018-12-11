Cloud-Native Workshop [![Build Status](https://travis-ci.org/openshift-labs/cloud-native-guides.svg?branch=ocp-3.10)](https://travis-ci.org/openshift-labs/cloud-native-guides)
===
This one day hands-on cloud-native workshops provides developers and introduction to cloud-natives applications and gives them an experience of building cloud-native applications using OpenShift, Spring Boot, WildFly Swarm, Vert.xt and more.

Agenda
===
* Introduction to Cloud-Native apps
* Building services with Spring Boot
* Deploying Java EE services based on WildFly Swarm
* Deploying Reactive Services based on Vert.x
* Deploying Web UI based on Node.js
* Monitoring Application Health
* Fault Tolerance and Service Resilience
* Configuration Management 
* Continuous Delivery 
* Debugging Services


Install Workshop Infrastructure
===

An [APB](https://github.com/mcouliba/cloud-native-development-apb) is provided for 
deploying the Cloud-Native Workshop infra (lab instructions, Nexus, Gogs, Eclipse Che, etc) in a project 
on an OpenShift cluster via the service catalog. In order to add this APB to the OpenShift service catalog, log in 
as cluster admin and follow the instructions in [Cloud Native Development APB](https://github.com/mcouliba/cloud-native-development-apb) project.

Or if you have Ansible installed locally, you can also run the Ansilbe playbooks directly on your machine:

```
oc login
oc new-project lab-infra

ansible-playbook -vvv playbooks/provision.yml \
       -e namespace=$(oc project -q) \
       -e openshift_token=$(oc whoami -t) \
       -e openshift_master_url=$(oc whoami --show-server)
``` 

Lab Instructions on OpenShift
===

Note that if you have used the above workshop installer, the lab instructions are already deployed.

```
$ oc new-app osevg/workshopper:latest --name=guides \
    -e WORKSHOPS_URLS="https://raw.githubusercontent.com/mcouliba/cloud-native-guides/ocp-3.10/_cloud-native-workshop-che.yml"
$ oc expose svc/guides
```

Local Lab Instructions
===
```
$ docker run -it -p 8080:8080 \
      -v $(pwd):/app-data \
      -e CONTENT_URL_PREFIX="file:///app-data" \
      -e WORKSHOPS_URLS="file:///app-data/_cloud-native-workshop-che.yml" \
      osevg/workshopper:latest
```
