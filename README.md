
# Multi container application

The project uses a simplified topic (Fibonacci calculator) as an implementation just because we don't need to focus on the actual implementation details of the application, but rather focus on Docker and the deployment side. The purpose of this tiny web application is to create a CI/CD process for building, testing and deploying a multi-container application in AWS.
Let's give some brief information about the configuration

Below is overall architecture of the web application:
![Architecture](https://github.com/calliduss/multi-docker/blob/master/figures/Prod.PNG)


Project structure (only deployment side): 

```
 .travis.yml 
 docker-compose.yml  
 Dockerrun.aws.json 
├───client 
│   │   Dockerfile 
│   │   Dockerfile.dev 
│   │
│   └───nginx 
│          default.conf 
│
├───nginx 
│       default.conf 
│       Dockerfile 
│       Dockerfile.dev 
│
├───server 
│      Dockerfile 
│      Dockerfile.dev 
│
└───worker 
       Dockerfile 
       Dockerfile.dev 
```


The picture below shows CI/CD process:
![Architecture](https://github.com/calliduss/multi-docker/blob/master/figures/cicd.PNG)


## Description

The web application consists of 4 containers:

- client
- server
- worker
- nginx

Each module contains two dockerfiles:

* Dockerfile.dev
* Dockerfile

Dockerfile is a configuration that defines:
* How our container behaves
* What different programms it's going to contain
* What is startup command

Dockerfile flow: 
* Specify a base image (`FROM`)
* Select the default working directory. Create if doesn't exist (`WORKDIR`)
* Run some commands to install additional programs/dependencies (`RUN/COPY`)
* Specify a command to run on container startup (`CMD`)

[Dockerfile cheat sheet](https://design.jboss.org/redhatdeveloper/marketing/docker_cheatsheet/cheatsheet/images/docker_cheatsheet_r3v2.pdf)

The purpose of Dockerfile.dev is to run tests on startup in Travis CI. The other is used for deployment in production (AWS)

### Docker compose

Docker compose is a tool for defining and running multi-container Docker applications, whereas Dockerfiles are simple text files that contain the commands to assemble an image that will be used to deploy containers. Compose allows define the services that make up your app in docker-compose.yml so they can be run together in an isolated environment.

The project contains only one docker-compose.yml

### NGINX

Nginx module additionally includes default.conf file for routing management. 
The client module has its own nginx for routing


### Travis CI

Travis CI is a hosted continuous integration service used to build and test software projects hosted at GitHub and Bitbucket.
Travis CI is integrated with Github but it's necessary to sync (or resync) with your Github account
.travis.yml file tells Travis CI what to do

-> before_install step builds test version of React project 
-> script uses an image that was created in before_install step and run tests against it 
-> after_success step builds production version of all projects and pushes them to Docker hub 

Its necessary to specify environment variables Travis CI that are used in .travis.yml
https://travis-ci.org/github/calliduss/multi-docker/settings

![Architecture](https://github.com/calliduss/multi-docker/blob/master/figures/env_vars.PNG)


### AWS

A Dockerrun.aws.json file is an Elastic Beanstalk–specific JSON file that describes how to deploy a set of Docker containers or a multicontainer Docker environment as an Elastic Beanstalk application. Dockerrun.aws.json describes the containers to deploy to each container instance (Amazon EC2 instance that hosts Docker containers) in the environment as well as the data volumes to create on the host instance for the containers to mount.

[AWS configuration cheat sheet](?)
