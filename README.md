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
    - Use the `.pem` format to allow login from the terminal

#### EC2 instances

In the `userdata` directory, there are a set of shell scripts that can be used to provision each EC2 instance. The details of each provisioning script can be explored further [here](https://github.com/fahimtq1/vprofile-multi-tier-project), where every command in each script is defined step-by-step.

The order in which the EC2 instances will be provisioned is as follows:

- MySQL instance (database)
- Memcached instance
- RabbitMQ instance
- Tomcat instance

Steps:

- Each of these instances use a CentOS 7 system (apart from the Tomcat instance- Ubuntu 18.04 OS is used), which can be found in the AWS Marketplace when choosing the AMI
- Instance type: t2.micro
- Key pair: created key pair
- Select existing security group: choose the relevant security group for the instance being made
- Configure storage: 8GiB gp2 Root volume
- Advanced details: paste the relevant provisioning script in the `User data` section for the instance being made

#### Route 53

Once each backend service instance is running, the private IP addresses of these instances have to be updated in a Route 53 private hosted zone (domain) for these backend services.

Repeat these steps to make a record for each backend service:

- Give the domain a relevant name and description
- Select private hosted zone
- Select create hosted zone
- In this private hosted zone, select create record
- Provide a record name that relates to the backend service
- Select record type A- Routes traffic to an IPv4 address and some AWS resources
- In Value, paste the private IPv4 address of the backend service instance
- Select a Simple routing policy
- Select create record

#### Building and deploying the artifact

There are a few dependencies required on the localhost for this stage and to install them on a Windows machine [chocolatey](https://chocolatey.org/docs/installation) needs to be installed. Then the following commands can be run on the localhost CLI:

- `choco install jdk8`
- `choco install maven`
- `choco install awscli`

Building the artifact:

- `git clone https://github.com/fahimtq1/vprofile-cloud-project.git`- clone this repository onto the localhost
- `cd src/main/resources`- navigate to the application.properties file location
- `nano application.properties`- edit the addresses of the backend services to the names of the records that have been created in the private hosted zone:

![application properties](https://user-images.githubusercontent.com/99980305/203525617-4d0661b3-a1bc-493d-be16-293f7e478c22.png)

- `mvn install`- run this command in the pom.xml location
- `ls target`- validate the installation by checking if vprofile-v2.war (the artifact) is present in the target directory

- Navigate to the IAM Console on AWS to add an IAM User to allow S3 programmatic access
    
    - Select Users
    - Select Add users
    - Give a relevant name
    - Select AWS credential type - Access key- Programmatic access
    - Select Next
    - Select Attach existing policies directly - AmazonS3FullAccess policy

- Once the IAM User has been created, an Access key ID and a Secret access key- these will be used to configure awscli
- `aws configure`- input this in the CLI and paste the correct credential information
- `aws s3 mb s3://bucket-name`- creates an S3 bucket
- `cd target`- navigate to the target directory
- `aws s3 cp vprofile-v2.war s3://bucket-name/vprofile-v2.war`- copies the artifact into the S3 bucket with the same name
- `aws s3 ls s3://bucket-name`- validates the copy by listing the objects inside the bucket

- Navigate to the IAM Console on AWS to add an IAM Role to give S3 bucket access to an EC2 instance

    - Select Roles
    - Select Create role
    - Trusted entity type - AWS service
    - Use case - EC2
    - AmazonS3FullAccess permission policy
    - Give a relevant name and select Create role

- Attach the IAM Role to the Tomcat (application) EC2 instance

    - Navigate to the EC2 console and select the app instance
    - Actions - Security - Modify IAM role
    - Select the created IAM role and then Update IAM role

- Connect to the app instance via SSH from the localhost CLI
- `sudo -i`- gives root user permissions 
- `systemctl status tomcat8`- validate the service
- `cd /var/lib/tomcat8/webapps`- navigate to the webapps directory of the tomcat8 service
- `systemctl stop tomcat8`- stop the service before making configuration changes
- `rm -rf ROOT`- removes the default application
- `cd`- navigate to the home location
- `apt install awscli`- doesn't need to be configured, as the EC2 instance has permissions from the attached IAM role
- `aws s3 ls s3://bucket-name`- view the objects in the S3 bucket
- `aws s3 cp S3://bucket-name/vprofile-v2.war /tmp/vprofile-v2.war`- copy the artifact from the S3 bucket to the EC2 instance /tmp directory
- `cp /tmp/vprofile-v2.war /var/lib/tomcat8/webapps/ROOT.war`- the artifact is copied into the tomcat8 service directory as the new default application
- `systemctl start tomcat8`- restarts the service with the new default application
- `cat /var/lib/tomcat8/webapps/ROOT/WEB-INF/classes/application.properties`- validate the application.properties file 
- `apt install telnet`- package used to validate the network connectivity of the instances
- `telnet db01.vprofile.in 3306`- this step can be repeated for the record of each backend service with their respective port

#### Load balancer

- Navigate to the EC2 console 

    - Select Target Groups on the sidebar
    - Select Create target group
    - Basic configuration - Instances
    - Give a relevant target group name
    - Advanced health check settings - Override - 8080 - Healthy threshold 2
    - Select Next
    - Register targets - app instance 
    - Select include as pending
    - Create target group

- Navigate to the EC2 console

    - Select Load Balancers on the sidebar
    - Select Create load balancer
    - Select Application Load Balancer
    - Give a relevant name
    - Select Internet-facing
    - Select all the Availability Zones under Network mapping- this allows the application to be highly available
    - Security groups - Select the security group for the load balancer
    - Listener and routing - Add listener - HTTPS 443 - Select target group
    - Select the SSL/TLS certificate for the registered domain from ACM
    - Select create load balancer

- Copy the DNS name (this is the endpoint) of the load balancer from it's details tab
- Go to the domain provider and add a CNAME record, with the DNS name pasted, in the DNS management console

#### Autoscaling group

## Conclusion
