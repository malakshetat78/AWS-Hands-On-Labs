# Build a VPC and Launch a Web Server on AWS

## Lab overview

This lab builds a virtual private cloud (VPC) in N. Virginia (`us-east-1`), adds public and private subnets across two Availability Zones, and launches an Apache web server on Amazon EC2. The server runs in the second public subnet and accepts HTTP traffic through a dedicated security group.

**Target architecture.** The completed design places `Web Server 1` in the second public subnet. Its public route table sends internet-bound traffic to the internet gateway. The target private route sends internet-bound traffic to a NAT gateway in the first public subnet.

![Target VPC architecture with four subnets and an EC2 web server](./screenshots/Screenshot%202026-11-25%20123601.png)

**Starting architecture.** Task 1 starts with one public subnet and one private subnet in `us-east-1a`. Task 2 extends the VPC into `us-east-1b`.

![Initial VPC design with two subnets in the first Availability Zone](./screenshots/Screenshot%202026-11-25%20123618.png)

## Task 1 — Create the VPC

In the AWS console, switch to **N. Virginia (`us-east-1`)**. Open **VPC → Create VPC** and select **VPC and more**.

1. Set **Name tag auto-generation** to `lab` and the **IPv4 CIDR block** to `10.0.0.0/16`.

   ![VPC wizard showing the lab name prefix and IPv4 CIDR](./screenshots/Screenshot%202026-11-25%20120942.png)

2. Select one Availability Zone (`us-east-1a`), one public subnet, and one private subnet. Under **Customize subnets CIDR blocks**, set the public subnet to `10.0.0.0/24` and the private subnet to `10.0.1.0/24`.

   ![Initial public and private subnet CIDR settings](./screenshots/Screenshot%202026-11-25%20120955.png)

3. For the target lab design, configure one zonal NAT gateway in `us-east-1a`. Set **VPC endpoints** to **None**, and enable both **DNS hostnames** and **DNS resolution**. Choose **Create VPC**, then check that `lab-vpc` appears as **Available** under **Your VPCs**.

   The wizard screenshots show **None** selected for NAT gateways. The final NAT gateway configuration is unverified.

   ![VPC endpoint and DNS settings in the wizard](./screenshots/Screenshot%202026-11-25%20121005.png)

   ![The lab-vpc entry marked Available](./screenshots/Screenshot%202026-11-25%20121717.png)

## Task 2 — Add subnets and configure routing

### Create subnets in the second Availability Zone

Open **VPC → Subnets → Create subnet** and select `lab-vpc` for both subnets. Use the following values.

| Subnet name | Availability Zone | IPv4 CIDR block |
| --- | --- | --- |
| `lab-subnet-public2` | `us-east-1b` | `10.0.2.0/24` |
| `lab-subnet-private2` | `us-east-1b` | `10.0.3.0/24` |

The CIDR field screenshot shows the form before the lab subnet CIDR was entered. Use the `/24` values in the table.

![Subnet CIDR input before entering the lab value](./screenshots/Screenshot%202026-11-25%20121445.png)

![Create subnet form for lab-subnet-public2](./screenshots/Screenshot%202026-11-25%20121528.png)

![Create subnet form for lab-subnet-private2](./screenshots/Screenshot%202026-11-25%20121633.png)

![Both additional subnets marked Available in lab-vpc](./screenshots/Screenshot%202026-11-25%20121704.png)

### Associate the private and public route tables

1. Under **Route tables**, open `lab-rtb-private1-us-east-1a`. For the target design, verify in **Routes** that `0.0.0.0/0` points to the NAT gateway. Under **Subnet associations → Edit subnet associations**, select `lab-subnet-private1-us-east-1a` and `lab-subnet-private2`, then save.

   ![Both private subnets selected for association](./screenshots/Screenshot%202026-11-25%20122041.png)

2. Open `lab-rtb-public` and verify in **Routes** that `0.0.0.0/0` points to the internet gateway. Under **Subnet associations → Edit subnet associations**, select `lab-subnet-public1-us-east-1a` and `lab-subnet-public2`, then save.

   ![Both public subnets selected for association](./screenshots/Screenshot%202026-11-25%20122153.png)

   The captures show subnet selections and CIDR blocks. The route destinations are validation steps for reproducing the lab.

## Task 3 — Create the web security group

In **VPC → Security groups → Create security group**, name the group `Web Security Group`, enter `Enable HTTP access` as the description, and select `lab-vpc`. Add an inbound **HTTP** rule on **TCP port 80** from **Anywhere-IPv4** (`0.0.0.0/0`) with the description `Permit web requests`.

![Web Security Group creation form and inbound HTTP rule](./screenshots/Screenshot%202026-11-25%20122435.png)

If the outbound rules list is empty, the prior lab walkthrough specifies allowing **All traffic** to **Anywhere-IPv4** (`0.0.0.0/0`) so the instance can fetch packages and application files. The creation confirmation shows one outbound permission entry, with its settings outside the capture.

![Security group creation confirmation and inbound HTTP rule on TCP port 80](./screenshots/Screenshot%202026-11-25%20122530.png)

## Task 4 — Launch and test the EC2 web server

1. Open **EC2 → Launch instance**. Name the instance `Web Server 1`, choose **Amazon Linux 2023**, select `t2.micro` and the lab key pair `vockey`, and keep the lab's default 8 GiB `gp3` root volume.

   ![EC2 launch form showing Web Server 1 and Amazon Linux](./screenshots/Screenshot%202026-11-25%20122859.png)

   ![t2.micro instance type and vockey key pair](./screenshots/Screenshot%202026-11-25%20122908.png)

2. Under **Network settings**, select `lab-vpc` and `lab-subnet-public2`. Enable **Auto-assign public IP** and choose the existing `Web Security Group`.

   ![EC2 networking settings showing the second public subnet and existing web security group](./screenshots/Screenshot%202026-11-25%20122916.png)

3. Under **Advanced details → User data**, enter the lab script. It installs the packages, downloads and extracts the sample application into Apache's document root, and starts Apache.

   ```bash
   #!/bin/bash
   dnf install -y httpd wget php mariadb105-server
   wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-ACCLFO-2/2-lab2-vpc/s3/lab-app.zip
   unzip lab-app.zip -d /var/www/html/
   chkconfig httpd on
   service httpd start
   ```

   ![EC2 user data field containing the lab bootstrap script](./screenshots/Screenshot%202026-11-25%20122924.png)

4. Choose **Launch instance**. In **EC2 → Instances**, wait for `Web Server 1` to reach **Running** and pass its status checks. Copy its **Public DNS** and open it in a browser over `http://`.

   ![EC2 summary showing Web Server 1 running in the second public subnet with public DNS](./screenshots/Screenshot%202026-11-25%20123510.png)

5. Check that the sample application displays instance metadata. In the browser capture, the instance ID matches the EC2 summary and the app reports `us-east-1b`.

   ![Sample application displaying EC2 instance metadata and Availability Zone us-east-1b](./screenshots/Screenshot%202026-11-25%20123452.png)

## Conclusion

The console and browser captures show `Web Server 1` running in the second public subnet and the sample application displaying metadata for that instance. The lab covers VPC addressing, subnet placement across Availability Zones, route table associations, security group rules, EC2 user data, and browser-based verification.
