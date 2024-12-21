# Automating-EC2-Instance-Configuration-and-Application-Deployment-with-Ansible
Automating EC2 Instance Configuration and Application Deployment with Ansible

Automating EC2 Configuration with Ansible

In this guide, we will walk through setting up a simple Ansible environment on two EC2 instances running a Linux-based operating system. This will automate the installation of Apache (httpd), the deployment of a GitHub repository to the web server, and the configuration of SSH key-based authentication for passwordless login.

## Objectives
By the end of this tutorial, you will be able to:
- Set up Ansible on a master EC2 instance (`ansible-master`).
- Configure passwordless SSH access between the master and slave EC2 instances.
- Write an Ansible playbook to install Apache and clone a GitHub repository on a remote EC2 instance (`slave-server`).
- Test the setup by accessing the deployed application in a web browser.

## What is Ansible?
Ansible is an open-source automation tool designed for tasks such as configuration management, application deployment, and infrastructure orchestration. It provides a simple yet powerful way to manage multiple systems simultaneously without requiring agents on the target machines.

---

## Step 1: Launch Two EC2 Instances

1. **Create EC2 Instances**:
   - Launch two EC2 instances (e.g., `ansible-master` and `slave-server`) using Amazon Linux 2 or another Linux distribution.
   - Assign roles to the instances:
     - `ansible-master`: The Ansible control node to run Ansible commands.
     - `slave-server`: The target server where the playbook will be applied.
   - Ensure both instances can communicate over port 22 (SSH).
   - Note the public IP addresses of both instances for SSH access and inventory configuration.

---

## Step 2: Set Up Ansible on the Master EC2 Instance

### Update the System
Run the following on both `ansible-master` and `slave-server`:
```bash
sudo yum update -y
```

### Install Ansible (on `ansible-master` only):
```bash
sudo yum install ansible -y
```

### Generate SSH Key for Passwordless Authentication
1. Generate an SSH key pair on `ansible-master`:
   ```bash
   ssh-keygen -t rsa -b 2048 -f ~/.ssh/ansible_key
   ```

2. Copy the public key to `slave-server`:
   - Copy the content of `~/.ssh/ansible_key.pub` from `ansible-master`.
   - Append it to the `~/.ssh/authorized_keys` file on `slave-server`.

---

## Step 3: Prepare the Ansible Environment

### Create an Ansible Directory
On `ansible-master`, create a directory to store Ansible files:
```bash
mkdir ~/ansible
cd ~/ansible
```

### Configure the Inventory File
Create an inventory file to define the target hosts:
```bash
echo "[webservers]\n<slave-server-private-ip>" > inventory
```
Replace `<slave-server-private-ip>` with the private IP of your `slave-server` instance.

### Write the Ansible Playbook
Create a playbook file `playbook.yml` with the following content:
```yaml
---
- name: Install Apache and Deploy GitHub Repository
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

    - name: Test and enable Apache service
      block:
        - name: Test Apache configuration
          command: httpd -t
          register: httpd_config_test
          failed_when: httpd_config_test.stderr != "Syntax OK"

        - name: Start and enable Apache
          service:
            name: httpd
            state: started
            enabled: true

  handlers:
    - name: Restart httpd
      service:
        name: httpd
        state: restarted
```
Replace the repository URL with your GitHub repository if needed.

---

## Step 4: Run the Ansible Playbook

Run the playbook from `ansible-master`:
```bash
ansible-playbook -i inventory playbook.yml
```
This command will:
- Connect to the `slave-server` as defined in the inventory.
- Execute the tasks defined in `playbook.yml`.
- Install Apache and Git, clone the repository, and start the Apache service.

---

## Step 5: Verify the Web Application

1. **Check Apache Status**:
   On `slave-server`, run:
   ```bash
   ssh -i ~/.ssh/ansible_key ec2-user@<slave-server-public-ip>
   sudo systemctl status httpd
   ```
   Ensure that the Apache service is active and running.

2. **Access the Application**:
   - Open a web browser.
   - Enter the public IP address of `slave-server`.
   - Verify that the index page of your GitHub repository is displayed.

---

## Step 6: Troubleshooting

1. **Web Page Not Loading**:
   - Check the security group settings of `slave-server` to ensure HTTP (port 80) is open.
   - Update the inbound rules for the security group in the AWS EC2 console.

2. **Apache Not Starting**:
   - Verify the logs on `slave-server` for errors.
   - Ensure the repository was cloned correctly and that Apache configuration files are valid.

---

## Conclusion
In this tutorial, we set up an Ansible environment with two EC2 instances: one as the Ansible control node (`ansible-master`) and the other as the target server (`slave-server`). Using an Ansible playbook, we automated the installation of Apache, cloned a GitHub repository, and deployed it to the web server.

Ansible simplifies server management, enabling scalable and efficient automation for configurations and deployments across multiple systems. This workflow can be extended for more complex deployments, saving time and minimizing errors.




