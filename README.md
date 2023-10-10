# Automate EC2 provisioning in AWS using Jenkins and Ansible Playbook
![Jenkins-Ansible](https://github.com/rutikdevops/DevOps-Project-9/assets/109506158/f2df6d5c-723f-447c-a581-18fe02997331)

# Project Blog link :-

# Project Overview :-
- We will learn how to create new EC2 instances using the Ansible playbook and automate using Jenkins Pipeline. in the end, we will play the game of 2048.

# Project Steps :-

# 1. Launch an Ubuntu(22.04) T2 medium Instance:-
- Launch an AWS T2 medium Instance. Use the image as Ubuntu. You can create a new key pair or use an existing one. Enable HTTP and HTTPS settings in the Security Group.
<img width="960" alt="image" src="https://github.com/rutikdevops/DevOps-Project-9/assets/109506158/c69ab020-9ac7-4be6-a705-92a0fa970585">

# 2. Install Jenkins and Trivy:-
- To Install Jenkins :- Connect to your console, and enter these commands to Install Jenkins
```bash
vi jenkins.sh
```

```bash
#!/bin/bash
sudo apt update -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
                  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
                  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
                              /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```

```bash
sudo chmod 777 jenkins.sh
./jenkins.sh    # this will installl jenkins
```

- Once Jenkins is installed, you will need to go to your AWS EC2 Security Group and open Inbound Port 8080, since Jenkins works on Port 8080.
- Now, grab your Public IP Address

```bash
<EC2 Public IP Address:8080>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
<img width="555" alt="image" src="https://github.com/rutikdevops/DevOps-Project-9/assets/109506158/4223bac5-8721-4d1c-b89d-92e1a476045d">

- Unlock Jenkins using an administrative password and install the suggested plugins.
- Jenkins will now get installed and install all the libraries.
- Create a user click on save and continue.
- Jenkins Getting Started Screen.



# 3. Install Trivy:-
```bash
vi trivy.sh
```

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```

# 4. Install Ansible :-

- connect to your Jenkins machine using Putty or Mobaxtreme
- Now we are going to run the below commands on the Jenkins machine

```bash
sudo apt-get update                                                  # Update your system packages
sudo apt install software-properties-common                          # First Install Required packages to install Ansible
sudo add-apt-repository --yes --update ppa:ansible/ansible           # Add the ansible repository via PPA
sudo apt install python3                                             # Install Python3 on Jenkins for Ansible
sudo apt install ansible -y                                          # Install Ansible on Ubuntu 22.04 LTS
sudo apt install ansible-core                                        # Copy
ansible --version                                                    # To check version
sudo apt install python3-pip -y                                      # Install Python-pip3
sudo pip3 install boto boto3                                         # Install Boto Framework - AWS SDK
sudo apt-get install python3-boto -y                                 # Ansible will access AWS resources using Boto SDK.
pip list boto | grep boto
```

# 5. let's create and attach an IAM role to Jenkins machines for the provision of a new ec2 instance
- Navigate to AWS CONSOLE --> Click the "Search" field --> Type "IAM enter" --> Click "Roles" --> Click "Create role" --> Click "AWS service" --> Click "Choose a service or use case" --> Click "EC2" --> Click "Next" --> Click the "Search" field. --> Add permissions policies --> AmazonEC2FullAccess --> click Next --> Click the "Role name" field. --> Type "Jenkins-cicd" --> Click "Create role" --> Click "EC2"

- go to the Jenkins instance and add this role to the Ec2 instance:-
  select Jenkins instance --> Actions --> Security --> Modify IAM role --> Add a newly created Role and click on Update IAM role.

- Let's go to the Jenkins machine and add the Ansible Plugin :-
  Manage Jenkins --> Plugins --> Available Plugins --> search for Ansible and install

- Give this command in your Jenkins machine to find the path of your ansible which is used in the tool section of Jenkins.
```bash
which ansible
```

- Copy that path and add it to the tools section of Jenkins at ansible installations.
<img width="945" alt="image" src="https://github.com/rutikdevops/DevOps-Project-9/assets/109506158/2e4da92a-06d5-4811-96ec-a446c8e3c780">



# 6. Ansible Playbook :-
```bash
---
- name: Provisioning a new EC2 instance and security group
  hosts: localhost
  connection: local
  gather_facts: False
  tags: provisioning

  pre_tasks:

    - name: Gather facts
      setup:

    - name: Print python version
      debug:
        msg: "Using Python {{ ansible_python_version }}"

    - name: Install dependencies
      shell: "/usr/bin/python3.10 -m pip install {{ item }}"
      loop:
       - boto3
       - botocore

  vars:
    ansible_python_interpreter: /usr/bin/python3.10
    keypair: Project-1                                       # Set your key pair instead of Project-1
    instance_type: t2.micro
    image_id: ami-0f5ee92e2d63afc18
    wait: yes
    group: webserver
    count: 1
    region: ap-south-1
    security_group: ec2-security-group
    tag_name:
      Name: Rutik-EC2                                        # If you want to change your instance name change it from here

  tasks:
    - name: Create a security group
      amazon.aws.ec2_group:
        name: "{{ security_group }}"
        description: Security Group for webserver Servers
        region: "{{ region }}"
        rules:
          - proto: tcp
            from_port: 22
            to_port: 22
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            from_port: 8080
            to_port: 8080
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            from_port: 3000
            to_port: 3000
            cidr_ip: 0.0.0.0/0  
          - proto: tcp
            from_port: 80
            to_port: 80
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            from_port: 443
            to_port: 443
            cidr_ip: 0.0.0.0/0
        rules_egress:
          - proto: all
            cidr_ip: 0.0.0.0/0   
      register: basic_firewall

    - name: Launch the new EC2 Instance
      amazon.aws.ec2_instance:
        security_group: "{{ security_group }}"
        instance_type: "{{ instance_type }}"
        image_id: "{{ image_id }}"
        wait: "{{ wait }}"
        region: "{{ region }}"
        key_name: "{{ keypair }}"
        count: "{{ count }}"
        tags: "{{ tag_name }}"
        user_data: |
          #!/bin/bash
          sudo apt update -y
          sudo apt install docker.io -y
          sudo systemctl start docker
          sudo systemctl enable docker
          sudo docker run -d --name 2048 -p 3000:3000 sevenajay/2048:latest
      register: ec2
```
- tag_name: Set your key pair instead of Project-1
- keypair: If you want to change your instance name change it from here


- Write a sample pipeline for Provision
```bash
pipeline {
    agent any
    tools{
        ansible 'ansible'
    }
    stages {
        stage('cleanws') {
            steps {
                cleanWs()
            }
        }
        stage('checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/Aj7Ay/ANSIBLE.git'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }    
        stage('ansible provision') {
          steps {
             // To suppress warnings when you execute the playbook    
             sh "pip install --upgrade requests==2.20.1"
             ansiblePlaybook playbook: 'ec2.yaml' 
            }
        }
    }
}
```

# 7. Stage view :-
<img width="960" alt="image" src="https://github.com/rutikdevops/DevOps-Project-9/assets/109506158/463e4594-ccf3-4783-be05-292b0d17b54e">

- Provision Ec2-instance
<img width="959" alt="image" src="https://github.com/rutikdevops/DevOps-Project-9/assets/109506158/d997790b-28fb-43d8-a8b7-ad50c986c18a">


- copy the Public IP of the provisioned instance
```bash
<public-ip:3000>
```

- Play Game 2048
![image](https://github.com/rutikdevops/DevOps-Project-9/assets/109506158/3b72504c-020b-4aa4-8e37-15b5fcb334ef)



# Project Reference :-
- https://youtu.be/57uKMBL1DB4?feature=shared
- https://mrcloudbook.hashnode.dev/automate-ec2-provisioning-in-aws-using-jenkins-and-ansible-playbook

