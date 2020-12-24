
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

* `Dockerfile.dev`
* `Dockerfile`

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

The purpose of `Dockerfile.dev` is to run tests on startup in Travis CI. The other is used for deployment in production (AWS)

### Docker compose

Docker compose is a tool for defining and running multi-container Docker applications, whereas Dockerfiles are simple text files that contain the commands to assemble an image that will be used to deploy containers. Compose allows define the services that make up your app in docker-compose.yml so they can be run together in an isolated environment.

The project contains only one `docker-compose.yml`

### NGINX

Nginx module additionally includes `default.conf` file for routing management. 
The client module has its own nginx for routing


### Travis CI

Travis CI is a hosted continuous integration service used to build and test software projects hosted at GitHub and Bitbucket.
Travis CI is integrated with Github but it's necessary to sync (or resync) with your Github account
`.travis.yml` file tells Travis CI what to do

- `before_install` step builds test version of React project 
- `script` uses an image that was created in before_install step and run tests against it 
- `after_success` step builds production version of all projects and pushes them to Docker hub 

Its necessary to specify environment variables Travis CI that are used in `.travis.yml`
https://travis-ci.org/github/calliduss/multi-docker/settings

![Architecture](https://github.com/calliduss/multi-docker/blob/master/figures/env_vars.PNG)


### AWS

A `Dockerrun.aws.json` file is an Elastic Beanstalk–specific JSON file that describes how to deploy a set of Docker containers or a multicontainer Docker environment as an Elastic Beanstalk application. `Dockerrun.aws.json` describes the containers to deploy to each container instance (Amazon EC2 instance that hosts Docker containers) in the environment as well as the data volumes to create on the host instance for the containers to mount.

<details>
  <summary>AWS configuration cheat sheet</summary>

### EBS Application Creation

1. Go to AWS Management Console and use Find Services to search for Elastic Beanstalk
2. Click “Create Application”
3. Set Application Name to 'multi-docker'
4. Scroll down to Platform and select Docker
5. *In Platform Branch, select Multi-Container Docker running on 64bit Amazon Linux*
6. Click Create Application
7. You may need to refresh, but eventually, you should see a green checkmark underneath Health.


### RDS Database Creation

1. Go to AWS Management Console and use Find Services to search for RDS
2. Click Create database button
3. Select PostgreSQL
4. In Templates, check the Free tier box.
5. Scroll down to Settings.
6. Set DB Instance identifier to *multi-docker-postgres*
7. Set Master Username to *postgres*
8. Set Master Password to *postgrespassword* and confirm.
9. Scroll down to Connectivity. Make sure VPC is set to Default VPC
10. Scroll down to Additional Configuration and click to unhide.
11. Set Initial database name to *fibvalues*
12. Scroll down and click Create Database button


### ElastiCache Redis Creation

1. Go to AWS Management Console and use Find Services to search for ElastiCache
2. Click Redis in sidebar
3. Click the Create button
4. *Make sure Cluster Mode Enabled is NOT ticked*
5. In Redis Settings form, set Name to multi-docker-redis
6. Change Node type to 'cache.t2.micro'
7. Change Replicas per Shard to 0
8. Scroll down and click Create button


### Creating a Custom Security Group
1. Go to AWS Management Console and use Find Services to search for VPC
2. Find the Security section in the left sidebar and click Security Groups
3. Click Create Security Group button
4. Set Security group name to multi-docker
5. Set Description to multi-docker
6. Make sure VPC is set to default VPC
7. Click Create Button
8. Scroll down and click Inbound Rules
9. Click Edit Rules button
10. Click Add Rule
11. Set Port Range to 5432-6379
12. Click in the box next to Source and start typing 'sg' into the box. Select the Security Group you just created.
13. Click Create Security Group


### Applying Security Groups to ElastiCache

1. Go to AWS Management Console and use Find Services to search for ElastiCache
2. Click Redis in Sidebar
3. Check the box next to Redis cluster
4. Click Actions and click Modify
5. Click the pencil icon to edit the VPC Security group. Tick the box next to the new multi-docker group and click Save
6. Click Modify


### Applying Security Groups to RDS

1. Go to AWS Management Console and use Find Services to search for RDS
2. Click Databases in Sidebar and check the box next to your instance
3. Click Modify button
4. Scroll down to Network and Security and add the new multi-docker security group
5. Scroll down and click Continue button
6. Click Modify DB instance button


### Applying Security Groups to Elastic Beanstalk

1. Go to AWS Management Console and use Find Services to search for Elastic Beanstalk
2. Click Environments in the left sidebar.
3. Click MultiDocker-env
4. Click Configuration
5. In the Instances row, click the Edit button.
6. Scroll down to EC2 Security Groups and tick box next to multi-docker
7. Click Apply and Click Confirm
8. After all the instances restart and go from No Data to Severe, you should see a green checkmark under Health.


### Add AWS configuration details to .travis.yml file's deploy script

1. Set the region. The region code can be found by clicking the region in the toolbar next to your username.
eg: 'us-east-1'
2. app should be set to the EBS Application Name
eg: 'multi-docker'
3. env should be set to your EBS Environment name.
eg: 'MultiDocker-env'
4. Set the bucket_name. This can be found by searching for the S3 Storage service. Click the link for the elasticbeanstalk bucket that matches your region code and copy the name.
eg: 'elasticbeanstalk-us-east-1-923445599289'
5. Set the bucket_path to 'docker-multi'
6. Set access_key_id to $AWS_ACCESS_KEY
7. Set secret_access_key to $AWS_SECRET_KEY


### Setting Environment Variables

1. Go to AWS Management Console and use Find Services to search for Elastic Beanstalk
2. Click Environments in the left sidebar.
3. Click MultiDocker-env
4. Click Configuration
5. In the Software row, click the Edit button
6. Scroll down to Environment properties
7. In another tab Open up ElastiCache, click Redis and check the box next to your cluster. Find the Primary Endpoint and copy that value but omit the :6379
8. Set REDIS_HOST key to the primary endpoint listed above, remember to omit :6379
9. Set REDIS_PORT to 6379
10. Set PGUSER to postgres
11. Set PGPASSWORD to postgrespassword
12. In another tab, open up the RDS dashboard, click databases in the sidebar, click your instance and scroll to Connectivity and Security. Copy the endpoint.
13. Set the PGHOST key to the endpoint value listed above.
14. Set PGDATABASE to fibvalues
15. Set PGPORT to 5432
16. Click Apply button
17. After all instances restart and go from No Data, to Severe, you should see a green checkmark under Health.


### IAM Keys for Deployment

You can use the same IAM User's access and secret keys from the single container app we created earlier.


### AWS Keys in Travis

1. Go to your Travis Dashboard and find the project repository for the application we are working on.
2. On the repository page, click "More Options" and then "Settings"
3. Create an AWS_ACCESS_KEY variable and paste your IAM access key
4. Create an AWS_SECRET_KEY variable and paste your IAM secret key


### Deploying App

1. Make a small change to your src/App.js file in the greeting text.
2. In the project root, in your terminal run:

```
git add.
git commit -m “testing deployment"
git push origin master

```
3. Go to your Travis Dashboard and check the status of your build.
4. The status should eventually return with a green checkmark and show "build passing"
5. Go to your AWS Elasticbeanstalk application
6. It should say "Elastic Beanstalk is updating your environment"
7. It should eventually show a green checkmark under "Health". You will now be able to access your application at the external URL provided under the environment name.
</details>
