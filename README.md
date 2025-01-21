# Frappe Deployment on EC2 with MariaDB RDS (3-tier Architecture)

## Prerequisites

1. **Create a Custom VPC**:
   - Include 2 public and 2 private subnets.
   - Use the VPC wizard to create route tables, Internet Gateway (IGW), NAT Gateway, endpoints, etc.

2. **Create an RDS Instance in a Private Subnet**:
   - Engine: MariaDB 10.11
   - Availability Zones: 2
   - Choose the created VPC and private subnets.
   - Set `Publicly Accessible` to `false`.

3. **Create a Load Balancer**:
   - Use HTTP:80 and HTTPS:443 listeners.
   - If you have a domain and SSL certificate, use the 443 listener.
   - If you don't have a domain, use the 80 listener.
   - Create a target group (TG) during the ALB setup with HTTP:8000 and select the target with 80 in pending state, then click create.

4. **Create an EC2 Instance**:
   - OS: Ubuntu 22 AMI (reason: running `ssm-agent`).
   - Optionally create two instances: one in a private subnet and one in a public subnet for jump access.
   - Choose the created VPC and one of the private subnets.
   - Use the default Security Group (SG).
   - Instance Type: Minimum `t3.medium`.

5. **Configure Security Groups**:
   - Inbound rules for ALB SG:
     ![ALB Security Group Inbound Rules](https://github.com/user-attachments/assets/a19975bd-aaad-4f59-a1ae-e5d5e631803c)

   - Inbound rules for EC2 SG:
     ![EC2 Security Group Inbound Rules](https://github.com/user-attachments/assets/2684bb90-d115-4ab9-818f-6574aea13e58)

   - Target Group configuration:
     ![Target Group Configuration](https://github.com/user-attachments/assets/eb4dafdb-8bc7-4ff4-8552-fd562b1f01c5)

   - Load Balancer listener configuration:
     ![Load Balancer Listener](https://github.com/user-attachments/assets/efc6d605-b15f-4519-9e3d-d1f0e37ca140)

## Deployment Steps

1. **Login to the Private Machine and Execute the Following Commands**:

   ```bash
   sudo su -
   sudo apt-get update
   python3 --version
   sudo apt install git python3-dev python3-pip redis-server
   sudo apt install software-properties-common
   sudo apt-get install mariadb-client-10.6
   sudo curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
   export NVM_DIR="$HOME/.nvm"
   [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
   nvm install 18
   node -v
   npm install -g yarn
   sudo apt-get install xvfb libfontconfig wkhtmltopdf
   sudo pip3 install frappe-bench
   bench --version
   sudo apt install python3.10-venv
   bench init frappe-bench --frappe-branch version-15
   cd frappe-bench/
   ssh-keygen
   bench get-app erpnext https://github.com/frappe/erpnext --branch version-15 --resolve-deps
   bench config restart_supervisor_on_update on
   sudo bench setup production ubuntu
   bench new-site prod-frappe-alb-1920024901.ap-south-1.elb.amazonaws.com --db-host=prod-frappe-mariadb.cu4mn4dld2td.ap-south-1.rds.amazonaws.com --db-port=3306 --db-username=root
   bench --site prod-frappe-alb-1920024901.ap-south-1.elb.amazonaws.com install-app erpnext
   bench --site prod-frappe-alb-1920024901.ap-south-1.elb.amazonaws.com list-apps
   bench setup nginx
   sudo service nginx reload
   bench setup add-domain prod-frappe-alb-1920024901.ap-south-1.elb.amazonaws.com
   lsof -i :8000
   bench build
   chmod o+rx /home/ubuntu/
   ```
The content in command like prod-frappe-alb-1920024901.ap-south-1.elb.amazonaws.com is alb dns, You must use the alb dns if you want you site to be accessible from any browser. The site_name, alb dns and added-domain should be exactly same.

This ubuntu user is firstly add to sudo group, use command 'usermod -aG sudo ubuntu'

Addition Config are done in creating tg

Frappe Accessible using ALB
![{425755CE-EE1F-49ED-886F-B34AF60F63A9}](https://github.com/user-attachments/assets/385b65c5-f468-4c60-9148-bd660d3dbbcd)

