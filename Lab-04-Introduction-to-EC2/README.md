# Introduction to Amazon EC2

This lab deploys an Apache web server on Amazon EC2 and covers instance monitoring, HTTP access, resizing, service quotas, and stop protection.

![Lab architecture diagram showing an EC2 instance inside a security group and an Availability Zone](images/ec2-lab-architecture.png)

*Lab reference diagram. The diagram labels the web server as IIS. This walkthrough uses Apache HTTP Server on Amazon Linux 2023.*

**Result.** After restarting as a `t2.small`, the instance passed both EC2 status checks and served **Hello From Your Web Server!** at its new public IPv4 address.

## 1. Launch the web server

Launch an instance named **Web Server** using the following lab settings.

| Setting | Lab configuration |
| --- | --- |
| AMI | Amazon Linux 2023 |
| Instance type | `t2.micro` |
| Key pair | `vockey` |
| Network | Lab VPC / PublicSubnet1 |
| Termination protection | Enable under **Advanced details** |

Under **User data**, enter the following script to install Apache, enable it at boot, start it, and create the test page.

```bash
#!/bin/bash
dnf install -y httpd
systemctl enable httpd
systemctl start httpd
echo '<html><h1>Hello From Your Web Server!</h1></html>' > /var/www/html/index.html
```

**Evidence.** The EC2 console shows **Web Server** running as a `t2.micro` with `2/2 checks passed`. The console capture and system log also show Amazon Linux 2023 and `vockey`. Termination protection is a launch instruction whose saved state is unverified.

![Web Server running as t2.micro with both EC2 status checks passed](images/instance-running.png)

## 2. Monitor the instance

1. In the EC2 console, open **Instances** and select **Web Server**. Confirm that the instance is **Running** and that **Status check** shows **2/2 checks passed**. The instance list in Step 1 shows this result.

2. Open the **Monitoring** tab. Review **CPU utilization**, **Network in**, and **Network out**. The screenshot shows metrics for the selected one-hour window with detailed monitoring disabled.

   ![EC2 Monitoring tab with CPU utilization, network traffic, and detailed monitoring disabled](images/monitoring.png)

3. Select **Actions** → **Monitor and troubleshoot** → **Get system log**. Review the boot output for cloud-init messages. The captured log shows Apache package installation and the symlink created when the `httpd` service was enabled.

   ![EC2 system log showing Apache packages and the httpd service symlink](images/monitoring-system-log.png)

4. Select **Actions** → **Monitor and troubleshoot** → **Get instance screenshot**. Inspect the console capture for the operating system and boot state. The screenshot shows the **Amazon Linux 2023** login prompt.

   ![EC2 instance screenshot showing the Amazon Linux 2023 login prompt](images/monitoring-instance-console.png)

## 3. Allow HTTP and test the page

The first browser request timed out while the web server’s security group had no inbound rules. Edit the group to allow **HTTP** on **TCP port 80** from **Anywhere-IPv4** (`0.0.0.0/0`) and save the rule. Open the instance’s public IPv4 address over HTTP.

![Inbound rule editor showing HTTP on TCP port 80 from Anywhere-IPv4](images/http-rule.png)

**Verified result.** The browser displayed the expected Apache test page before the resize.

![Browser displaying the Apache test page before the resize](images/web-before-resize.png)

## 4. Resize the instance and root volume

Stop **Web Server** and change its instance type from `t2.micro` to `t2.small`. In the root EBS volume’s **Modify volume** screen, change the size from **8 GiB** to **10 GiB**, then start the instance and retest HTTP using its current public IPv4 address.

**Storage evidence.** The **Storage** tab showed an attached 8 GiB root volume. The modification form shows 10 GiB entered for that volume. The applied volume size and filesystem capacity are unverified.

![Modify volume form showing the 10 GiB target for the gp3 volume](images/ebs-size.png)

**Verified compute result.** After restart, **Web Server** was running as a `t2.small` with `2/2 checks passed`. The page loaded at the new public IPv4 address.

![Web Server running as t2.small with both EC2 status checks passed](images/instance-resized.png)

![Browser displaying the Apache test page after the restart](images/web-after-restart.png)

## 5. Review EC2 service quotas

In **Service Quotas**, select **Amazon Elastic Compute Cloud (Amazon EC2)** and filter for `running on-demand`. The console shows the matching quotas and their applied account-level values, AWS defaults, and utilization.

![Amazon EC2 service quotas filtered for Running On-Demand](images/service-quotas.png)

## 6. Test stop protection

With stop protection enabled, a stop attempt was rejected. The console identified the `disableApiStop` instance attribute.

![EC2 console rejecting the stop request while stop protection is enabled](images/stop-blocked.png)

After clearing **Enable** in **Change stop protection**, the next stop request was accepted and the instance entered **Stopping**. The final **Stopped** state was unverified.

![EC2 console accepting the stop request and showing the instance as Stopping](images/stop-initiated.png)

## Submission

Submission and grading are unverified in the supplied screenshots.
