# How to Securely Create and Use a Database on AWS

## Create an RDS instance

The first step is to create an RDS instance. This is assuming you already have an EC2 instance spun up.

1. Login to AWS and go to **RDS** under the Database section.
2. Click on **Create Database**
3. Select the database engine you would like to use (we are using *PostgreSQL*) and click **Next**
4. Under use case, choose **Dev/Test** to stay in the *Free Usage Tier* and click **Next**
5. Set your DB details in the next page.
    * License model: **postgresql-license**
    * DB engine version: **PostgreSQL 10.4-R1**
    * DB instance class: **db.t2.micro** - _this one is free_
    * Storage type: **General Purpose (SSD)**
    * Allocated storage: **20** - _min/max for free tier_
    * DB Instance Identifier: **{Enter a name for your instance}**
    * Master username: **{Enter username for database}**
    * Master password: **{Enter password for user}**
6. Click **Next** and configure advanced settings
    * Virtual Private Cloud (VPC): **{Choose the same VPC as your EC2 instance}**
      * If you just created your EC2 instance the default VPC in this box will be the right one.
    * Subnet group: **{default}**
    * Public accessibility: **No** - _only hosts on the VPC can access. very important!!_
    * Availability zone: **No preference** - _unless of course you have a preference_
    * VPC security groups: **Create new VPC security group**
    * Database name: **{Enter a name for your database}**
    * Database port: **5432** - _in this case, default for PostgreSQL_
    * DB parameter group: **{Should default to the right one}**
    * Encryption: _free tier does not support encryption so we can't do anything_
    * Backup: _leave this fields as is or change per your preference_
    * Monitoring: _again, up to you_
    * Maintenance: _set these up depending on what you prefer, they are fine as default_
7. Click **Create Database** and wait for your database to be created!

## Configure your DB's new security group

Now, we need to alter the security settings of the database instance we just created to allow the EC2
instance to connect to it.

1. Go to the **RDS Console**
2. Click on **Instances**
3. Find and click on the instace you just created.
4. Scroll down and click on the security group under **Connect**
    * It will look like **rds-launch-wizard...***
5. You are now in the EC2 Management Console Security Group page with your group ID selected.
6. Click on **Inbound** in one of the lower tabs.
7. Click on **Edit** to change the rule.
8. There should be an existing rule with **Type** `PostgreSQL` - we need to change the **Source**
9. Change the **Source** to `Anywhere` and click **Save**
    * Now we can access this Database from **anywhere** *within* the **VPC**.
    * The **VPC** is basically a private network within AWS.

## Create an SSH tunnel and connect to the database from your machine

An SSH tunnel creates a port on your machine and maps it to a port on a remote machine.

1. The tunnel is created by the following command:
    * `ssh -N -L {**your local port**}:{**RDS DB ADDRESS**}:5432 {**EC2 Username**}@{**EC2 Address**} -i {**your key file**}`
      * Your local port is the port on your machine we will map the connection to.
      * The RDS DB ADDRESS can be found within the RDS Console under **Connect**
      * We are using 5432 as that is the default port for PostgreSQL.
2. The tunnel is running! Now connect with psql.
    * If you do not have the `psql` command available, you will need to install PostgreSQL on your machine.
3. `psql -h 127.0.0.1 -p 5433 -U {username} -d {database name}`
    * `127.0.0.1` is the IP address of your local machine
    * `-p` specifies which port to use, we are using 5433, which was binded above
    * Use the **username** and **database name** that you created in the beginning.
    * You will be prompted for a password, enter the one you created in the beginning.
4. That's it! You are now connected to your cloud database instance using an SSH tunnel. Make sure you close the tunnel after you are done by pressing `CTRL + C`



