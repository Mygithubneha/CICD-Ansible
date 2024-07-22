Introduction

In today’s fast-paced world, the ability to quickly and efficiently deploy and manage infrastructure is crucial for the success of any project. In this blog, we will explore how to automate the configuration of Nginx and the deployment of applications using Ansible and Jenkins. Additionally, we will dive into the process of setting up a robust and secure MongoDB cluster using Ansible roles. By the end of this blog, you will have a comprehensive understanding of how to build a production-ready infrastructure that can be easily replicated and scaled.

Objective

The main objective of this blog is to demonstrate how automation can simplify the setup and management of critical components of a web application stack. By utilizing Ansible and Jenkins, we aim to achieve the following:

Configure Nginx Server: Learn how to automate the installation and configuration of Nginx, a high-performance web server, to serve as a reverse proxy for our web applications.
Deploy Applications with Ansible and Jenkins: Understand the integration of Ansible and Jenkins to automate the deployment process, ensuring consistent and error-free application releases.
Configure a Production-Ready MongoDB Cluster: Explore the steps to set up a MongoDB cluster with Ansible roles, ensuring high availability and fault tolerance.
Prerequisites

Before diving into the automation process, readers should have some familiarity with the following concepts:

Basic knowledge of Ansible: Understanding Ansible playbooks, roles, and inventory is beneficial for better comprehension.
Working knowledge of Jenkins: Familiarity with Jenkins and its CI/CD capabilities will be advantageous.
Web Servers: Basic knowledge of web servers like Nginx will help grasp the configuration process.
MongoDB fundamentals: Understanding the basics of MongoDB and database clusters will be beneficial when setting up the MongoDB cluster.
For more info about Ansible Roles, Please refer to the link: Roles — Ansible Documentation

Step by Step implementation guide:

Ansible Master: This Server will run the playbooks on the remote server through Jenkins jobs.

Nginx Server(2): Both Nginx Servers are in different az for high availability purposes.

MongoDB Server(2): Both Servers are in different az for high availability purposes with no public ip as this is the database.

To Create Ansible Master, Click on ‘Launch Instances’.


Give the name to the server and I am using Ubuntu 22.04 version which is one of the prerequisites for this project.


I am using Instance type t2.small because We have to install Ansible and Jenkins on the same machine. To get rid of the issue, select t2.small instead of t2.micro.


Networking settings must be in your bucket.

Don’t open all the traffic. As this is the worst practice of being a DevOps Engineer.

Here, I want to connect with my Ansible-Master through my local machine only. So, According to that, I am providing my local machine's public IP address.

You can find your public IP by going to https://whatismyipaddress.com/ or you can run the command ‘ curl ifconfig.me ‘ and click on ‘Launch Instance.


Here, We need to configure our Security group for our Ansible-Master Server.

We need to open an 8080 port for Jenkins and 22 ports are already opened for SSH.

Make sure to provide your public IP only as a source instead of opening all traffic as we have already talked about this(security/networking) above.


Now, log in to the Ansible-Master Server.


First thing, Install the Ansible by following the below commands.

#! /bin/bash
apt install software-properties-common
add-apt-repository ppa:deadsnakes/ppa
apt install python3.11 -y
sudo update-alternatives - install /usr/bin/python python /usr/bin/python3.11 1
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible -y
sudo apt update
ansible - version

After Installing Ansible, We have to automate our configurations by Jenkins.

To Install Jenkins, you can follow the below commands.

#! /bin/bash
# For Ubuntu 22.04
# Intsalling Java
apt update -y
apt install openjdk-11-jre -y
java - version
# Installing Jenkins
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y

Let’s do the setup of Jenkins first.

Enter the password by going to the highlighted directory.


We will go with ‘Install suggested plugins’ because we have to install our required plugins according to the project.


Install two plugins,

Ansible: To integrate with Ansible and make the configuration deployment automated.

Build Pipeline: To create a pipeline and make the configuration deployment pipeline.


Now, Set the Credentials by following the below process:

On the Ansible-Master run command ‘ssh-keygen -t rsa -b 4096’.


Go to the .ssh/ directory and copy the content of the private key which is named id_rsa.


On Jenkins, Add the private key by creating global credentials. To do that, Go to Dashboard > Manage Jenkins and Go to Credentials


Click on System


Click on ‘+ Add credentials’


Select the Kind as ‘SSH Username with private key’, and provide the ID(mandatory for Jenkins's point of view) and username as well.


Paste the content of the private key of Ansibe-Master and click on Create.


Here, you have added the global credentials.


Let’s create a pipeline to automate the Ansible configuration deployment.

Give the name and select pipeline.


Here, As we don’t know the code that how to integrate Ansible with Jenkins then we will see through the Pipeline Syntax.


Now, provide the parameters according to your need such as the playbook name and hosts files.


Click on ‘Generate Pipeline Script’ to generate the script for it.


Here, you can see the script that I have got according to my provided parameters.


I have created a Jenkins pipeline according to my requirements. You can also do that according to your requirements and push it on GitHub or GitLab.


Here, I am providing the Jenkinsfile that is in the GitHub repository, and click on Save.

https://github.com/Mygithubneha/CICD-Ansible.git


After Creating Pipeline, we need servers where we have to run our Ansible playbooks.

First of all, we will configure Nginx Servers.

To do that, Create an instance with the below configuration.

Make sure Both Nginx Servers should be created one by one and for different az to make high availability.



Make sure the 22 port should be open for Ansible-Master Server. As We are following DevOps best practices and for port 80 you can open all the traffic because we are providing our content to the open world.


Local to Ansible Master

Now, we can connect with our Nginx Server through the Ansible-Master server as we have provided Ansible-Server Master private ip in the Nginx Server security group.

To do that send the pem file that you have selected while creating the Nginx Servers to the Ansible Server from the local machine.

You can use the below command to do that.


After doing that, we have to copy the public key that we generated earlier on Ansible-Master. To connect with the Nginx Servers, Ansible server should have something in common.

To do that, We are going to put the public key in the authorized_keys of the Nginx Servers.

Copy the id_rsa.pub file from the Ansible-Master server and paste it into the Nginx Servers inside the .ssh/authorized_keys file.


After doing this for both Nginx Servers. We have to provide the private IP to Ansible. So, whenever any playbook will start running. So, Ansible will get to know that on this server the playbook needs to run.

To do that, open the /etc/ansible/hosts file and write the following things according to your Private IPs and the rest of the things remain the same.


After doing all the setup, we run our Jenkins pipeline and it is successfully completed.


As you can see our web application is running perfectly on both servers without any issues.



Conclusion

Automation is a game-changer in modern infrastructure management, and with the power of Ansible and Jenkins, we have successfully demonstrated how to build a production-ready infrastructure. By streamlining the configuration of Nginx, we can ensure a reliable and scalable environment for hosting web applications.

Through this blog, we hope to empower readers to harness the capabilities of automation and implement best practices in their own projects. By adopting automation, organizations can not only save time and effort but also reduce the likelihood of human errors in their infrastructure.

And the blog on setting up a production-ready MongoDB cluster can be accessed here:

Automating Infrastructure: Ansible Playbooks for Nginx and MongoDB Configuration: Chapter 2

Let’s meet there and complete our Project in the next blog.