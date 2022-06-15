# Assignment for Ansible Advanced Udemy Course
This is an Ansible playbook to deploy a distributed web app with Flask, MySQL and AWS EC2. This is solution assignment for the "Ansible Advanced" Udemy Course.

## Architechture
The web app is deployed in tiers, using the following components:
- Three EC2 `t2.micro` instances with `Amazon Linux 2` AMI
    - Two as web servers using Flask
        - On top of each server Flask is installed
        - Also the source code is cloned into `/opt/simple-webapp/` and then the app is started to run on port 5000
        - This web servers are assigned a security group enabling both TCP ports 22 and 5000 for inbound requests from the ELB 
    - One as MySQL server using MariaDB
        - MariaDB is installed and configured on this server. Also an example entry is added.
        - This server is assigned a security group enabling both TCP ports 22 and 3306 for inbound requests from web servers
- One Classic ELB to distribute load across the two Flask web instances
    - The load balancer is assigned a security group to enable inbound request from port 80
    - The load balancer maps request from port 80 to port 5000 on both Flask instances

## Details
### Prerequisites
1. An AWS IAM user with programatic access enabled (`aws_access_key` and `aws_secret_key` are needed)
2. An EC2 key-pair created and saved on the region where the instances are going to be deployed

### Configuration
There's an Ansible configuration file to set the inventory file and disable host key checking to avoid issues regarding SSH fingerprint at runtime.

### Inventory
The invetory consist of three main components:
- `master` which corresponds to the host executing the playbook. 
- `web` group which is used for the web servers
- `db` group which is used for the MySQL servers

### Variables
- The `master` host has a `host_var` file that contains
    - The `ansible_host`
    - The AWS credentials (in this case `aws_access_key` and `aws_secret_key` from an IAM user are used). This are used along the playbook and roles to deploy the AWS components.
    - The general AWS information (such as region, VPC, key pair name, image IDs, etc)
- There's an extra var file under `vars/external` with the AWS credentials 
- Also both `web` and `db` groups set the default `ec2-user` and the keypair path.
- There's a file under `group_vars/db.yaml` that contains the variables related to the database name, user and password.

### Playbooks & Roles
The main playbook is `assignment.yaml` which depends on four roles:
- `flask_ec2_instance`: To deploy and create an EC2 instance and its associated SG. Also it adds the recently created instances to the `web` group.
- `mysql_ec2_instance`: To deploy and create an EC2 instance and its associated SG. Also it adds the recently created instances to the `db` group.
- `flask_web`: To configure the deployed EC2 instances with the python dependencies, clone the code repository and start the webapp.
- `mysql_db`: To configure the deployed EC2 instance with MariaDB, create the database and add a test entry.

Lastly, the main playbook itself creates the load balancer using the flask web instances.

## Usage
The following variables must be changed/filled:
- `host_vars/master.yaml` - All the variables to select region, credentials and other AWS variables
- `vars/external.yaml` - The two AWS credentials
- `inventory.ini` - The `ansible_ssh_private_key_file` variable on both the web and db groups to your current path and file

Once the variables are set, run the playbook with
```shell
$ ansible-playbook assignment.yaml
```