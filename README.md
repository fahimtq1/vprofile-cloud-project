# Multi-Tier Application Deployment on AWS

Take a look at the local setup of this application using virtualisation with Vagrant and VirtualBox [here](https://github.com/fahimtq1/vprofile-multi-tier-project)

This application was inspired by the devopshydclub repository found [here](https://github.com/devopshydclub/vprofile-project)

You can see the most used languages in this project below:

![Your Repository's Stats](https://github-readme-stats.vercel.app/api/top-langs/?username=fahimtq1&theme=blue-green)

## Project brief

This scenario is of a DevOps engineer who has been given a multi-tier application stack and has been tasked with deploying an application on the AWS Cloud. The DevOps engineer will manually configure and provision the cloud infrastructure, ready for application deployment. There is currently a desire to reduce the number of applications running on on-site servers to reduce OpEx (operational expenses). The main benefits of deploying and application on the cloud are high availability and scalability. To learn more about cloud computing and its benefits, click [here](https://github.com/fahimtq1/cloud_computing_basics).

## Application architecture

![vprofile-project-architecture](https://user-images.githubusercontent.com/99980305/199768482-3bb654c1-8a40-4352-8e86-49b4ae50875f.png)

Note- Instead of the NGINX service, we will be using an Application Load Balancer (routes HTTP/HTTPS traffic)

### Application services

- **NGINX/Load balancer(ALB)**- a web service used for load balancing and reverse proxying (in this case we will be using an application load balancer in place of the NGINX service)
- **Tomcat**- an application service used to run JAVA server pages that are based on web applications
- **RabbitMQ**- a message broker/queuing agent service (used as a dummy service in this project)
- **Memcached**- database caching service
- **MySQL**- SQL database

### Application architecture explained

These are the basic steps of the application workflow:

- Users send requests that are received by the load-balancing service
- The application load balancer is a load-balancing service that receives that allows the frontend to listen on port 80 and then routes the request to the application (Tomcat) server on port 8080
- The Tomcat server receives the request on port 8080 and communicates with the backend services, on their respective ports, to receive the application content, which can then be routed to the frontend so the users can view it

## Cloud architecture

![vprofile-cloud-architecture](https://user-images.githubusercontent.com/99980305/202738252-d0a9176b-7e8f-42fe-a734-449985724cbc.png)

### Cloud services

- EC2 instances
- Application Load Balancer
- Autoscaling Group
- Target Group
- Launch configuration
- AMI
- S3 bucket

## Cloud setup

### Plan

The basic flow of execution is as follows:

- Login to AWS
- Create key pair
- Create security groups for the EC2 instances and the load balancer
- Launch each EC2 instance with their respective user data
- Create records with the Route 53 service
- Update the IP addresses in the `application.properties` file with the names of the created records
- Build the application locally from the source code- this is an artifact
- Upload the artifact to an S3 bucket
- Download the artifact from the S3 bucket to the application (Tomcat) EC2 instances
- Setup the application load balancer with a HTTPS certificate from the ACM (AWS Certificate Manager)
- Map the load balancer endpoint to the website name in the GoDaddy DNS
- Build the autoscaling group for the Tomcat instances

### Steps

Note- prior to the project, a domain was created with a domain registrar and then given a SSL/TLS certificate using the ACM service to establish a HTTPS connection to the website. CNAME validation also had to be established. These steps will not be detailed.

#### Security groups and key pairs

For each security group, we will only be creating inbound rules 

- Security group for the load balancer- this will allow HTTPS and HTTP traffic from any IP address
    - Type HTTPS, Protocol TCP, Port range 443, Source any IPv4 and any IPv6
    - Type HTTP, Protocol TCP, Port range 80, Source any IPv4 and any IPv6
    - Type SSH, Protocol TCP, Port range 22, Source My IP

- Security group for the Tomcat instances- this will allow traffic from the load balancer on port 8080 
    - Type Custom TCP, Protocol TCP, Port range 8080, Source security group ID of the load balancer
    - Type SSH, Protocol TCP, Port range 22, Source My IP

- Security group for the backend services- this will give the application instances access to the backend services of the application
    - Type MYSQL/Aurora, Protocol TCP, Port range 3306, Source security group ID of the Tomcat instances
    - Type Custom TCP, Protocol TCP, Port range 5672, Source security group ID of the Tomcat instances
    - Type Custom TCP, Protocol TCP, Port range 11211, Source security group ID of the Tomcat instances
    - Type SSH, Protocol TCP, Port range 22, Source My IP

- Create key pair
    - Use the RSA encryption algorithm 
    - Use the `.pem` format to allow login from the local CLI

#### EC2 instances

#### Building and deploying the artifact

#### Load balancer 

#### Autoscaling group 

## Conclusion
