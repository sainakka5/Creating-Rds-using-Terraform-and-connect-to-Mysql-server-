PROJECT 2: Understand the below code & explain about it in the document, Also create a resource in AWS by using this code
STEPS TO CREATE AWS RDS :

1.	Create an EC2 instance in AWS console, I create an instance with name TERRAFORM_PROJ
2.	Login into Putty terminal with public -IP and keypair of that instance

   ![image](https://github.com/sainakka5/Creating-Rds-using-Terraform-and-connect-to-Mysql-server-/assets/136338958/0e5eb76a-352b-4a63-87da-3235c402779b)

3.	After login to centos root user, install Terraform and its supporting packages with linux script

        sudo yum update -y
        sudo yum install -y yum-utils
        sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo sudo yum -y install terraform

	After execution above commands, we installed Terraform and its supporting packages, from the above link we downloaded HASHICORP LANGUAGE packages

4.	first add our iam user details via>> export AWS_ACCESS_KEY_ID="your_access_key_id"
•	export AWS_SECRET_ACCESS_KEY="your_secret_access_key"
•	download awscli and give command "aws configure"
•	or create a  file with below script

        provider "aws" {
        region	= "us-west-2"
        access_key = "your_access_key_id" secret_key = "your_secret_access_key"
        }

	For access_key and secret_key we have to create a IAM user with suitable permission and generate access_key and secret_key for that IAM user
## create a file with name vpc.tf and give the below script to create of new vpc and save it ## 

    module "vpc" {	## This is the definition of the Terraform module named "vpc".
      source = "terraform-aws-modules/vpc/aws" ## This specifies the source of the module. In ## this case, it's using a community-provided module called terraform-aws- modules/vpc/aws#
      
      name = "sai_vpc"
      cidr = "192.31.0.0/16"
    
      azs	= ["ap-south-1a", "ap-south-1b", "ap-south-1c"]
      private_subnets = ["192.31.1.0/24", "192.31.2.0/24", "192.31.3.0/24"]
      public_subnets = ["192.31.101.0/24", "192.31.102.0/24", "192.31.103.0/24"]
    
      enable_nat_gateway = true
      enable_vpn_gateway = true
      
      tags = {
        Terraform = "true"
        Environment = "dev"
    }
    }
    resource "aws_security_group" "example_security_group" {
      name	= "example-security-group"
      description = "Example security group for VPC"
      vpc_id	= aws_vpc.sai_vpc.id
      // Inbound rules
     
      ingress {
        from_port = 22
        to_port	= 22
        protocol	= "tcp"
        cidr_blocks = ["0.0.0.0/0"] // Allow SSH traffic from all sources
      }
      ingress {
        from_port = 80
        to_port	= 80
        protocol	= "tcp"
        cidr_blocks = ["0.0.0.0/0"] // Allow HTTP traffic from all sources
      }
    }

5.	After installation create a directory with command mkdir<directory_name> and go to that directory with command cd < directory_name >. Which is act as module for this Terraform.
6.	Create a file name variable.tf and provide the variables for your RDS instance in a variables.tf file using HCL script mentioned below. Some variables that are included are the database engine, instance size, storage size, username and password, and backup retention period. Create that file with the command vi variable.tf. it automatically created and entered into that file by entering “I” (insert) the variables script mentioned in the below picture. I had entered my configurations in that script as variables. r, paste these keys in the variable file script and save it.
## to create an module create an directory and save main,variable,output files and save it, This directory name is called as MODULE ##

## To variable file save the below script to call values form this file ##

      variable "rds_instance_identifier" { ## Custom name of the DB instance (NOT a database name)
        description = "Custom name of the instance"
        default = "rds-instance"
      }
      
      
      variable "rds_is_multi_az" { ## Defaults to false. Set to true for a multi-az instance.
        description = "Set to true on production"
        default	= false
      }
       
      variable "rds_storage_type" { ## Defaults to standard (magnetic)
        description = "One of 'standard' (magnetic), 'gp2' (general purpose SSD), or 'io1' (provisioned IOPS SSD)."
        default	= "gp2"
      }
      
      
      variable "rds_iops" { ## if we give iops storage then we have to give this configure
        description = "The amount of provisioned IOPS. Setting this implies a storage_type of 'io1', default is 0 if rds storage type is not io1"
        default	= "0"
      }
      
      
      variable "rds_allocated_storage" { ## The number of GBs to allocate. Input must be an integer
        description = "The allocated storage in GBs"
        default = 10
      }
      
      
      variable "rds_engine_type" {	## Engine type, such as mysql or postgres
        description = "Database engine type"
        default	= "mysql"
      }
      
      
      variable "rds_engine_version" { ## type the version of yuor engine
        description = "Database engine version, depends on engine type"
        default	= "8.0"
      }
      
      
      variable "rds_instance_class" { ## instance size give the ram and no of cpu we have to give
        description = "Class of RDS instance"
        default	= "db.t2.micro"
      }
      
      
      variable "auto_minor_version_upgrade" { ## Automatically upgrade minor version of the DB (eg. from Postgres 9.5.3 to Postgres 9.5.4), default: true
       
        description = "Allow automated minor version upgrade"
        default	= true
      }
      
      
      variable "allow_major_version_upgrade" { ## Allow upgrading of major version of database (eg. from Postgres 9.5.x to Postgres 9.6.x), default: false
        description = "Allow major version upgrade"
        default	= false
      }
      
      
      variable "apply_immediately" { # Specifies whether any database modifications are applied immediately, or during the next maintenance window, default: false
        description = "Specifies whether any database modifications are applied immediately, or during the next maintenance window"
        default	= false
      }
      variable "maintenance_window" { ## The window to perform maintenance in. Syntax: 'ddd:hh24:mi- ddd:hh24:mi' UTC, default: "Mon:03:00-Mon:04:00"
        description = "The window to perform maintenance in. Syntax: 'ddd:hh24:mi-ddd:hh24:mi' UTC "
        default	= "Mon:03:00-Mon:04:00"
      }
      
      
      variable "database_name" { ## name of the dabatase
        description = "The name of the database to create"
        default	= "sai_database"
      }
      # Self-explainatory variables
      variable "database_user" { ## user name for your database
        description = "The username "
        default = "admin"
      }
      
      
      variable "database_password" { ## create an password for your user
        description = "The name of the database to create"
       
        default = "admin123"
      }
      variable "database_port" { ## define the port to your rds server
        default = 3306
      }
      
      
      variable "db_parameter_group" { ## Defaults to mysql5.6, for postgres postgres9.5
        description = "Parameter group, depends on DB engine used"
        default = "mysql8.0"
      }
      
      
      variable "use_external_parameter_group" { ## Defaults to false, if true use parameter group specified by parameter_group_name instead of a built-in one
        description = "Use parameter group specified by `parameter_group_name` instead of built-in one"
        default = false
      }
      
      
      # Use an external parameter group (i.e. defined in caller of this module) variable "parameter_group_name" {
        description = "Parameter group to use instead of the default"
        default	= "sai_parameter"
      }
      
      
      variable "publicly_accessible" { ## giving the public accessibility to the rds
        description = "Determines if database can be publicly available (NOT recommended)"
        default	= true
      }
      
      
      # RDS Subnet Group Variables ## give the ids of subnets that we created in the vpc.tf file variable "subnets" {
        description = "List of subnets DB should be available at. It might be one subnet"
        type	= list
        default = ["subnet-0da038fa8203c87c9","subnet-0b3e9d1b19e280f98","subnet-0a4584d1cc53cd084"]
      }
       
      variable "private_cidr" { ## List of CIDR netblocks for database security group, e.g. ["10.0.1.0/24", "10.0.2.0/24]
        description = "VPC private addressing, used for a security group"
        type = list
        default = ["192.31.1.0/24", "192.31.2.0/24", "192.31.3.0/24"]
      
      
      }
      variable "aws_security_group" { ## give the id of the security group created in vpc.tf file
        description = "ID of the existing security group"
        default = "sg-01a32eb575a3b10ce"
      }
      variable "rds_vpc_id" {  ## give the vpc id for this rdsinstance
        description = "VPC to connect to, used for a security group"
        default = "vpc-098b83ce158708696"
      
      
      }
      
      
      variable "skip_final_snapshot" { ## it gives the snapshot when we delete the rds for backup
        description = "If true (default), no snapshot will be made before deleting DB"
        default	= true
      }
      variable "copy_tags_to_snapshot" { ## copy all tags from RDS database to snapshot (default true)
        description = "Copy tags from DB to a snapshot"
        default	= true
      }
      
      
      variable "backup_window" { ## when to perform DB snapshot, default "22:00-03:00"; can't overlap with maintenance window
        description = "When AWS can run snapshot, can't overlap with maintenance window"
        default	= "22:00-03:00"
      }
      
      
      variable "backup_retention_period" { ## backup retention period in days (default: 0), must be > 0 to enable backups
       
        description = "How long will we retain backups"
        default	= 7
      }
      
      
      variable "tags" { ## A mapping of tags to assign to the DB instance
        description = "A map of tags to add to all resources"
        default	= {}
      }
      
      
      variable "monitoring_interval" { ## To disable collecting Enhanced Monitoring metrics, specify 0.
        description = "To disable collecting Enhanced Monitoring metrics, specify 0. The default is 0. Valid Values: 0, 1, 5, 10, 15, 30, 60."
        default	= "0"
      }

7.	Again create another with command “ vi main.tf ” and define the Terraform resources for your RDS instance. Use the aws_db_instance resource type to create the RDS instance and refer the values that we gave in variable.tf file with command var.<>, I had saved my configuration of my database in varible.tf file script. Save the script in the file.
	I had mentioned 3306 port in the security group of that instance which is default port of MYSQL engine

## to create the terra infrastructure create an main.tf file and save this script in it and call the values from the variable.tf file ##
 
    resource "aws_db_instance" "main_rds_instance" {
      identifier	= "${var.rds_instance_identifier}"
      allocated_storage = "${var.rds_allocated_storage}"
      engine	= "${var.rds_engine_type}"
      engine_version	= "${var.rds_engine_version}"
      instance_class	= "${var.rds_instance_class}"
      username	= "${var.database_user}"
      password	= "${var.database_password}"
    
      ## By giving var.<name of the configure mention in variable.tf file> it will call the value from that file ##
     
      port = "${var.database_port}"
     
      # Because we're assuming a VPC, we use this option, but only one SG id
      vpc_security_group_ids = ["${aws_security_group.main_db_access.id}"]
    
    
      # We're creating a subnet group in the module and passing in the name
      db_subnet_group_name = "${aws_db_subnet_group.main_db_subnet_group.id}"
      parameter_group_name = "${var.use_external_parameter_group ? var.parameter_group_name : aws_db_parameter_group.main_rds_instance[0].id}"
    
    
      # We want the multi-az setting to be toggleable, but off by default
      multi_az	= "${var.rds_is_multi_az}"
      storage_type	= "${var.rds_storage_type}"
      iops	= "${var.rds_iops}"
      publicly_accessible = "${var.publicly_accessible}"
    
    
      # Upgrades
      allow_major_version_upgrade = "${var.allow_major_version_upgrade}"
      auto_minor_version_upgrade = "${var.auto_minor_version_upgrade}"
      apply_immediately	= "${var.apply_immediately}"
      maintenance_window	= "${var.maintenance_window}"
    
    
      # Snapshots and backups
      skip_final_snapshot = "${var.skip_final_snapshot}"
      copy_tags_to_snapshot = "${var.copy_tags_to_snapshot}"
    
    
      backup_retention_period = "${var.backup_retention_period}"
      backup_window	= "${var.backup_window}"
    
    
      # enhanced monitoring
      monitoring_interval = "${var.monitoring_interval}"
    
    
      tags = tomap(merge(var.tags, { "Name" = format("%s", var.rds_instance_identifier) }))
    }
    
    
    resource "aws_db_parameter_group" "main_rds_instance" {
     
      count = "${var.use_external_parameter_group ? 0 : 1}"
    
    
      name = "${var.rds_instance_identifier}-${replace(var.db_parameter_group, ".", "")}-custom-params"
      family = "${var.db_parameter_group}"
    tags = tomap(merge(var.tags, { "Name" = format("%s", var.rds_instance_identifier) }))
    }
    
    
    resource "aws_db_subnet_group" "main_db_subnet_group" {
      name	= "${var.rds_instance_identifier}-subnetgrp"
      description = "RDS subnet group"
      subnet_ids = "${var.subnets}"
    
    
      tags = tomap(merge(var.tags, { "Name" = format("%s", var.rds_instance_identifier) }))
    }
    
    
    # Security groups
    resource "aws_security_group" "main_db_access" {
      name	= "${var.rds_instance_identifier}-access"
      description = "Allow access to the database"
      vpc_id	= "${var.rds_vpc_id}"
    
    
      tags = tomap(merge(var.tags, { "Name" = format("%s", var.rds_instance_identifier) }))
    }
    
    
    resource "aws_security_group_rule" "allow_db_access" {
      type = "ingress"
    
    
      from_port = "${var.database_port}"
      to_port	= "${var.database_port}"
      protocol	= "tcp"
      cidr_blocks = "${var.private_cidr}"
    
    
      security_group_id = "${aws_security_group.main_db_access.id}"
     
    }
    
    
    resource "aws_security_group_rule" "allow_all_outbound" {
      type = "egress"
    
    
      from_port = 0
      to_port	= 0
      protocol	= "-1"
      cidr_blocks = ["0.0.0.0/0"]
    
    
      security_group_id = "${aws_security_group.main_db_access.id}"
    }
    
    
    ## after creation of terraform to see the output of given lines iin the below , to just see the results of this rds ##
    
    
    output "rds_instance_id" {
      value = "${aws_db_instance.main_rds_instance.id}"
    }
    
    
    # Output the address (aka hostname) of the RDS instance output "rds_instance_address" {
      value = "${aws_db_instance.main_rds_instance.address}"
    }
    
    
    # Output endpoint (hostname:port) of the RDS instance output "rds_instance_endpoint" {
      value = "${aws_db_instance.main_rds_instance.endpoint}"
    }
    
    
    # Output the ID of the Subnet Group output "subnet_group_id" {
      value = "${aws_db_subnet_group.main_db_subnet_group.id}"
    }
     
    # Output DB security group ID output "security_group_id" {
      value = "${aws_security_group.main_db_access.id}"
    }

8.	To execute the scrips in the files, we have to run the following commands terraform init	# It downloads required plugins for that infrastructure terraform plan	# Print output, what it’s going to create
terraform apply # creates the infrastructure (RDS in this project)
Overall commands that are given in this instance are shown in the below figure
  
   ![image](https://github.com/sainakka5/Creating-Rds-using-Terraform-and-connect-to-Mysql-server-/assets/136338958/b7797d60-aa3f-457c-b798-07622391ef81)


9.	After creation of AWS RDS we will get endpoint which is like host name to connect the database to MYSQL workbench , go to the RDS >database>click on the created database there we get Endpoint address , copy that endpoint.


10.	Install and open the MYSQL workbench click on symbol of mysql connections then add all credentials to connect with our database
 
   ![image](https://github.com/sainakka5/Creating-Rds-using-Terraform-and-connect-to-Mysql-server-/assets/136338958/4cd422d5-a046-4b96-b599-4b47d60efb5a)


	Enter the details: Connection name: can be any; Hostname: endpoint of rds , username: one which we gave in “main.tf”file and after clicking on store in vault we can enter password the same which we gave in “main.tf”. Click on test connection and click on connection.

   ![image](https://github.com/sainakka5/Creating-Rds-using-Terraform-and-connect-to-Mysql-server-/assets/136338958/43a07274-c76f-44ec-ae41-5e7124c1aaae)


	You will enter into the mysql workbench sever with RDS connection as below, we can excecute sql commands in that workspace.

  ![image](https://github.com/sainakka5/Creating-Rds-using-Terraform-and-connect-to-Mysql-server-/assets/136338958/7b0090c6-bc96-41bf-9b98-ee239d25b4dd)

 
11.	By this we had verified that RDS instance is working correctly by connecting to it using a SQL client, such as MySQL Workbench or pgAdmin.


	By this project we will be able to demonstrate our ability to use Terraform to provision AWS resources, as well as we can understand of how RDS works and how to configure it using Terraform.
