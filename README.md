# Automating-EC2-Instance-Configuration-and-Application-Deployment-with-Ansible
Automating EC2 Instance Configuration and Application Deployment with Ansible

Introduction to Automating EC2 Configuration with Ansible
In this guide, we will walk you through setting up a simple Ansible environment on two EC2 instances running a Linux-based operating system. This will automate the installation of Apache (httpd) and deployment of a GitHub repository to the web server, along with the configuration of SSH key-based authentication for passwordless login.

By the end of this tutorial, you will be able to:

Set up Ansible on a master EC2 instance (ansible-master).

Configure passwordless SSH access between the master and slave EC2 instances.

Write an Ansible playbook to install Apache and clone a GitHub repository on a remote EC2 instance (slave-server).

Test the setup by accessing the deployed application in a web browser.

What is Ansible?
Ansible is an open-source automation tool designed for IT tasks such as configuration management, application deployment, and infrastructure orchestration. It provides a simple yet powerful way to manage multiple systems simultaneously without requiring the installation of agents on the target machines.

Step 1: Launch Two EC2 Instances
Create two EC2 instances (e.g., ansible-master and slave-server) using Amazon Linux 2 or any other Linux distribution.

ansible-master: This will be your Ansible control node, where you'll run Ansible commands.

slave-server: This will be your target server, where the playbook will be applied.

Ensure that both instances are in the same security group or are able to communicate over the required ports (especially port 22 for SSH).

Note down the public IP addresses of both instances for SSH access and inventory configuration later.



Step 2: Set Up Ansible on the Master EC2 Instance
Commands to run on both ansible-master and slave-server:
Update the system:


Copy

Copy
 sudo yum update -y
Install Ansible (on ansible-master):


Copy

Copy
 sudo yum install ansible -y


Generate SSH key for passwordless authentication:


Copy

Copy
 ssh-keygen -t rsa -b 2048 -f ~/.ssh/ansible_key


Copy the content of the generated public key (~/.ssh/id_rsa.pub) of ansible-master and ,


paste it into the ~/.ssh/authorized_keys file on slave-server.



Step 3: Prepare the Ansible Environment
Create an ansible directory to store your Ansible files:


Copy

Copy
 mkdir ~/ansible
 cd ~/ansible


Create an inventory file with the IP address of your slave-server. This file lists the hosts Ansible will manage.

Example inventory file (inventory):


Copy

Copy
 [webservers]
 <slave-server-private-ip>
Replace <slave-server-private-ip> with the actual private IP of your slave-server.

Create a playbook file (playbook.yml) to install Apache (httpd) and deploy a GitHub repository.

Example playbook (playbook.yml):


Copy

Copy
 ---
 - name: Install httpd and Deploy GitHub Repository
   hosts: all
   become: true

   tasks:
     - name: Install required packages
       yum:
         name:
           - httpd
           - git
         state: present

     - name: Clone GitHub repository
       git:
         repo: "https://github.com/samleti-balaji/Sign-Up-page.git"
         dest: /var/www/html
       notify:
         - Restart httpd

     - name: Test and enable httpd service
       block:
         - name: Test httpd configuration
           command: httpd -t
           register: httpd_config_test
           failed_when: httpd_config_test.stderr != "Syntax OK"

         - name: Start and enable httpd
           service:
             name: httpd
             state: started
             enabled: true

 handlers:
   - name: Restart httpd
     service:
       name: httpd
       state: restarted
Replace https://github.com/your-username/your-repo.git with the URL of your GitHub repository.

This playbook will install Apache (httpd), install git, clone the repository to /var/www/html, and ensure the Apache service is started and enabled.

Step 4: Run the Ansible Playbook
Run the Ansible playbook:

On the ansible-master server, execute the following command:


Copy

Copy
 ansible-playbook -i inventory playbook.yml
This command will:

Connect to the slave-server as defined in the inventory.

Run the tasks defined in the playbook.yml file.

Install Apache and Git, clone the repository, and start the Apache service.

Check the output of the playbook run. You should see something like:


Step 5: Verify the Web Application
Log into the slave-server and check if Apache is running:


Copy

Copy
 ssh -i ~/.ssh/ansible_key ec2-user@<slave-server-public-ip>
 sudo systemctl status httpd
Ensure that the status shows Apache is active and running.

Open the web browser and type the public IP of the slave-server:



You should see the index page of your GitHub repository, confirming that Apache is serving the web application correctly.

Step 6: Troubleshooting
If the page is not loading, check the security group settings of the slave-server to ensure that HTTP (port 80) is open to inbound traffic.

Go to the AWS EC2 console.

Select the slave-server instance and check its associated security group.

Make sure that the security group allows inbound HTTP traffic on port 80.

If Apache isn't starting, check the logs on the slave-server for errors:

Make sure the repository cloned correctly and that the Apache configuration files are valid.

Finall output you will see



Conclusion
In this blog post, we walked through the steps to set up an Ansible environment on two EC2 instances: one as the Ansible control node (ansible-master) and the other as the target server (slave-server). By using an Ansible playbook, we installed Apache, cloned a GitHub repository, and deployed it to the web server.

With Ansible, we simplified server management, making it easier to automate configurations and deployments on multiple servers. This workflow can be expanded for larger environments and more complex deployments, saving time and reducing the chances of human error.
