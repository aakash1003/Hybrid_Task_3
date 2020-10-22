# Hybrid_Task_3

Infrastructure as code using terraform, which automatically create a VPC on AWS



# Steps:

1. Write a Infrastructure as code using terraform, which automatically create a VPC.

2. In that VPC we have to create 2 subnets:

    a) public subnet [ Accessible for Public World! ]

    b) private subnet [ Restricted for Public World! ]

3. Create a public facing internet gateway for connect our VPC/Network to the internet world and attach    this gateway to our VPC.

4. Create a routing table for Internet gateway so that instance can connect to outside world, update and    associate it with public subnet.

5. Launch an ec2 instance which has Wordpress setup already having the security group allowing port 80      so that our client can connect to our wordpress site.

   Also attach the key to instance for further login into it.

6. Launch an ec2 instance which has MYSQL setup already with security group allowing port 3306 in       private subnet so that our wordpress vm can connect with the same.
   Also attach the key with the same.
   
  
   # Implementation:
   
   firstly, we create the profile, specify region and for plugins
   ```
   provider "aws" {
   region     = "ap-south-1"
   profile    = "aksharma"
   }
   ```
## Step 1: Now, we have to create VPC. For this we require CIDR block to specify range of IPv4 addresses for the VPC and a name tag for unique identification.
```
* Create VPC

resource "aws_vpc" "test-env" {
   cidr_block = "192.168.0.0/16"
   enable_dns_hostnames = true
   enable_dns_support = true
   tags ={
     Name = "test-env"
   }
}
```

## Step 2: In that VPC we have to create 2 subnets:
a) public subnet [ Accessible for Public World! ]

```
* public subnet

resource "aws_subnet" "public_subnet" {
   vpc_id = "${aws_vpc.test-env.id}"
   map_public_ip_on_launch = "true"
   cidr_block = "192.168.0.0/24"
   availability_zone = "ap-south-1a"
 }
 ```
b) private subnet [ Restricted for Public World! ]
```
* private subnet

resource "aws_subnet" "private_subnet" {
   vpc_id = "${aws_vpc.test-env.id}"
   map_public_ip_on_launch = "true"
   cidr_block = "192.168.1.0/24"
   availability_zone = "ap-south-1a"
 }
```

## Step 3 : Create a public facing internet gateway for connect our VPC/Network to the internet world and attach this gateway to our VPC.
```
* gateway

resource "aws_internet_gateway" "InterNetGateWay" {
  vpc_id = "${aws_vpc.test-env.id}"
  tags ={
    Name= "InternetGateWay"
  }
}
```

## Step 4: Create a routing table for Internet gateway so that instance can connect to outside world, update and associate it with public subnet.
```
* routing table

resource "aws_route_table" "public_route" {
  vpc_id = "${aws_vpc.test-env.id}"
route {
      cidr_block = "0.0.0.0/0"
      gateway_id = "${aws_internet_gateway.InterNetGateWay.id}"
    
}


resource "aws_route_table_association" "subnet_public" {
  subnet_id      = "${aws_subnet.public_subnet.id}"
  route_table_id = "${aws_route_table.public_route.id}"
}
```


## Step 5: Launch an ec2 instance which has Wordpress setup already having the security group allowing port 80 so that our client can connect to our wordpress site.
```
* security group(Wordpress)

resource "aws_security_group" "wp" {
  vpc_id = "${aws_vpc.test-env.id}"
  name        = "task2sg"
  
  ingress {
    description = "TCP"
    from_port   = 80	
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
}


  ingress {
     description = "SSH"
     from_port   = 22	
     to_port     = 22
     protocol    = "tcp"
     cidr_blocks = ["0.0.0.0/0"]


}
  egress {
     from_port   = 0	
     to_port     = 0
     protocol    = "-1"
     cidr_blocks = ["0.0.0.0/0"]
}  
```
This security group only allow ping, ssh and httpd.


Now we can create our instance. For creating any instance we need AMI,instance type,availability zone which we have mentioned earlier in subnet and key - here I used pre-existing key. For WordPress AMI, I used WordPress Base Version which requires subscription for that.
```
* WordPress Instance

resource "aws_instance" "wp_Instance" {
  ami           = "ami-000cbce3e1b899ebd"
  instance_type = "t2.micro"
  subnet_id = "${aws_subnet.public_subnet.id}"
  vpc_security_group_ids = ["${aws_security_group.wp.id}"]
  key_name = "taskkey"
 tags ={
    Name= "instance_wp"
  }
}
```


## Step 6: Launch an ec2 instance which has MYSQL setup already with security group allowing port 3306 in private subnet so that our wordpress vm can connect with the same.
```
* security group(MYSQL)

resource "aws_security_group" "mysql" {
  name = "MYSQL"
  description = "managed by terrafrom for mysql servers"
  vpc_id = "${aws_vpc.test-env.id}"
  ingress {
    protocol        = "tcp"
    from_port       = 3306
    to_port         = 3306
    security_groups = ["${aws_security_group.wp.id}"]
  }




 egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  
   }
}
```


### Creating Instance.
```
* MYSQL Instance

resource "aws_instance" "mysql_Instance" {
 ami           = "ami-08706cb5f68222d09"
 instance_type = "t2.micro"
 subnet_id = "${aws_subnet.private_subnet.id}"
 vpc_security_group_ids = ["${aws_security_group.mysql.id}"]
 key_name = "taskkey"
tags ={
   Name= "instance_mysql"
 }
}
```

## Then, Run the Terraform Code.
```
terraform init
```
## - Apply Terraform.
```
terraform apply -auto-approve
```
## - And at the end delete or destroy the complete process.
```
terraform destroy -auto-approve
```
### Thank You!!!
