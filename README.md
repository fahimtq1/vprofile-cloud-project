# Multi-Tier Application Deployment on AWS

Take a look at the local setup of this application using virtualisation with Vagrant and VirtualBox [here](https://github.com/fahimtq1/vprofile-multi-tier-project)

This application was inspired by devopshydclub repository found [here](https://github.com/devopshydclub/vprofile-project)

You can see the most used languages in this project below:

![Your Repository's Stats](https://github-readme-stats.vercel.app/api/top-langs/?username=fahimtq1&theme=blue-green)

## Project brief

The scenario is of a DevOps engineer who has been given a multi-tier application stack and has been tasked with deploying the application on a local machine. 

The application deals with a variety of services: web services, database services, application services etc.

The main challenge facing the DevOps engineer, regarding the local setup, is that the local setup is complex and time consuming. To solve this problem, the setup provisioning will be automated with Bash scripts and Virtual Machines will be configured with the use of Infrastructure as Code tools. 

First, the DevOps engineer will manually configure and provision the Virtual Machines, ready for application deployment. This is to ensure that the architecture network is properly configured, with any debugging handled before the automation step.

Finally, the entire setup will be automated. 

## Application architecture

![vprofile-project-architecture](https://user-images.githubusercontent.com/99980305/199768482-3bb654c1-8a40-4352-8e86-49b4ae50875f.png)

### Services

- **NGINX**- a web service used for load balancing and reverse proxying
- **Tomcat**- an application service used to run JAVA server pages that are based on web applications
- **RabbitMQ**- a message broker/queuing agent service (used as a dummy service in this project)
- **Memcached**- database caching service
- **MySQL**- SQL database

### Application architecture explained

These the basic steps of the application workflow:

- Users send requests that are received by the NGINX service
- NGINX is a load-balancing service that receives that allows the frontend to listen on port 80 and then routes the request to the app01 server on port 8080
- Tomcat receives the request on port 8080 and communicates with the backend services, on their respective ports, to receive the application content, which can then be routed to the frontend so the users can view it

## Cloud architecture

## Tools 

## Cloud setup 

## Conclusion