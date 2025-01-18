# Frappe Deployment on Ec2 with mariadb RDS (3-tier Arch)

Prerequisites:
1> create custom Vpc with 2 public and private Subnets.(use vpc and more option which will create rt, igw, nat, endpoint, etc)
2> Create Rds in private Subnet.
  Info- mariadb engine 10.11, 2 AZ, Choose Created VPC and private Subnet, Publically accessible=false, etc.
5> Create a Load Balancer with http:80 and https:443 listener, if you have the domain and SSL Certificate use 443 Listener, If you don't have the Domain use 80 Listener.Create a tg when creating alb with http:8000 and select the target with 80 in pending then click create.
4> Create an EC2 instance with ubuntu 22 ami( reason= running ssm-agent) or create 2 instances 1 in private and 1 in public to help you jump on private machine ), choose create vpc and one of the private subnet and select the default Sg, select minimum t3.medium type of instance and click create.
5> Here are some screen shoots that will help with sg inbounds rules,
This is the inbound rules of sg used for alb 
![{0C3A9908-BBB9-4C78-8436-FA6699C65258}](https://github.com/user-attachments/assets/a19975bd-aaad-4f59-a1ae-e5d5e631803c)

This is the inbound rules of sg used for EC2 
![{19FD2365-79FD-4DE5-8250-67EFBC52A3A7}](https://github.com/user-attachments/assets/2684bb90-d115-4ab9-818f-6574aea13e58)

The Target Group looks something like this 
![{EFFD8DFD-D8E4-4202-9669-6FFEAD9DFE2E}](https://github.com/user-attachments/assets/eb4dafdb-8bc7-4ff4-8552-fd562b1f01c5)

This is how the LB listener looks like
![{08FBA1C6-0A42-4211-A89A-A67AD7FE1FF3}](https://github.com/user-attachments/assets/efc6d605-b15f-4519-9e3d-d1f0e37ca140)

6> Now login to private machine and execute Below commands.

  

ubuntu@ip-10-23-44-118:~/frappe-bench$ history
    1  sudo su -
    2  sudo apt-get update
    3  python3 --version
    4  sudo apt install git python3-dev python3-pip redis-server
    5  sudo apt install software-properties-common
    6  sudo apt-get install mariadb-client-10.6
    7  sudo curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
    8  export NVM_DIR="$HOME/.nvm"
    9  [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
   10  nvm install 18
   11  node -v
   12  npm install -g yarn
   13  sudo apt-get install xvfb libfontconfig wkhtmltopdf
   14  sudo pip3 install frappe-bench
   15  bench --version
   16  sudo apt install python3.10-venv
   17  bench init frappe-bench --frappe-branch version-15
   18  cd frappe-bench/
   19  ls
   20  ssh-keygen
   21  bench get-app erpnext https://github.com/frappe/erpnext --branch version-15 --resolve-deps
   22  bench config restart_supervisor_on_update on
   23  sudo bench setup production ubuntu
   24  bench new-site prod-frappe-alb-1920024901.ap-south-1.elb.amazonaws.com --db-host=prod-frappe-mariadb.cu4mn4dld2td.ap-south-1.rds.amazonaws.com --db-port=3306 --db-username=root
   25  bench new-site prod-frappe-alb-1920024901.ap-south-1.elb.amazonaws.com --db-host=prod-frappe-mariadb.cu4mn4dld2td.ap-south-1.rds.amazonaws.com --db-port=3306 --db-root-username=root
   26  bench --site prod-frappe-alb-1920024901.ap-south-1.elb.amazonaws.com install-app erpnext
   27  bench --site prod-frappe-alb-1920024901.ap-south-1.elb.amazonaws.com list-apps
   28  bench setup nginx
   29  sudo service nginx reload
   30  bench setup add-domain prod-frappe-alb-1920024901.ap-south-1.elb.amazonaws.com
   31  lsof -i :8000
   32  bench build
   33  chmod o+rx /home/ubuntu/


* The content in command like prod-frappe-alb-1920024901.ap-south-1.elb.amazonaws.com is alb dns, You must use the alb dns if you want you site to be accessible from any browser. The site_name, alb dns and added-domain should be exactly same.

* This ubuntu user is firstly add to sudo group, use command 'usermod -aG sudo ubuntu'
* Addition Config are done in creating tg 
