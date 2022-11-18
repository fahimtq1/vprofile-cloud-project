# Multi-Tier Application Deployment on AWS

Take a look at the local setup of this application using virtualisation with Vagrant and VirtualBox [here](https://github.com/fahimtq1/vprofile-multi-tier-project)

This application was inspired by the devopshydclub repository found [here](https://github.com/devopshydclub/vprofile-project)

You can see the most used languages in this project below:

![Your Repository's Stats](https://github-readme-stats.vercel.app/api/top-langs/?username=fahimtq1&theme=blue-green)

## Project brief

This scenario is of a DevOps engineer who has been given a multi-tier application stack and has been tasked with deploying the application on the AWS Cloud. The DevOps engineer will manually configure and provision the cloud infrastructure, ready for application deployment. The main benefits of deploying and application on the cloud are high availability and scalability. To learn more about cloud computing and its benefits, click [here](https://github.com/fahimtq1/cloud_computing_basics).

## Application architecture

![vprofile-project-architecture](https://user-images.githubusercontent.com/99980305/199768482-3bb654c1-8a40-4352-8e86-49b4ae50875f.png)

### Services

- **NGINX**- a web service used for load balancing and reverse proxying
- **Tomcat**- an application service used to run JAVA server pages that are based on web applications
- **RabbitMQ**- a message broker/queuing agent service (used as a dummy service in this project)
- **Memcached**- database caching service
- **MySQL**- SQL database

### Application architecture explained

These are the basic steps of the application workflow:

- Users send requests that are received by the NGINX service
- NGINX is a load-balancing service that receives that allows the frontend to listen on port 80 and then routes the request to the app01 server on port 8080
- Tomcat receives the request on port 8080 and communicates with the backend services, on their respective ports, to receive the application content, which can then be routed to the frontend so the users can view it

## Cloud architecture

## Tools 

## Cloud setup 

## Conclusion