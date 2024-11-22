### Stage 1 - EC2 Only
#### Arch:
- Build an EC2 Instance.
- Build a detached IP Address.
- Build the security groups etc.
- Send the script from the terminal into Cloud Formation to build.
- Test this with ping checks.
- Build automatic and total removal of everything.
#### Software:
- Have WordPress installed via an Ansible playbook.
- Have the database installed via an Ansible playbook.
- Produce a bog standard WordPress site via Ansible playbook.
- Test by going to the IP Address built in the Arch.
#### DevOps:
- Turn the `wp-content` folder into a github repo.
- Add a step in the Ansible playbook for pulling the `wp-content` folder and creating a site.
- Pull repo locally, make some changes, push to GitHub and then pull using the Ansible playbook.
- Write some Python scripts that attempt to DDoS the server and crash it. To see what kind of performance different instances give you.
#### Just For Fun:
- If we get there, try and speed up the whole process of deployment.
- How fast can we make the deployment?
- Is it possible to do everything in under a minute?

### Stage 2 - EC2 + RDS
#### Arch:
- Build an EC2 Instance.
- Build a detached IP Address.
- Build an RDS instance for the database, simple, no failover, redundancy, etc.
- Build the security groups, in particular, lock down requests to RDS to come from EC2 only.
- Send the script from the terminal into Cloud Formation to build.
- Test this with ping checks.
- Build automatic and total removal of everything.
#### Software:
- Have WordPress installed via an Ansible playbook.
- Have the database installed via an Ansible playbook.
- Test by going to the IP Address built in the Arch.
#### DevOps:
- Add user data etc. to the site.
- Use Ansible to push that data into the RDS setup.
- Use Ansible to push the github repo of `wp-content`.
- Add some automated testing of the site using cypress or some other framework.
#### Just For Fun:
- If we get there, try and speed up the whole process of deployment.
- How fast can we make the deployment?
- Is it possible to do everything in under a minute?

### Stage 3 - EC2 + RDS (Hardcore)
#### Arch:
- Everything in stage 2.
- Add RDS in multiple availability zones.
- Add RDS failover.
- Add RDS snapshots for saving user data in case of failure.
- Add literally any other measures that will help data recovery.
- Add encryption, and GDPR compliant management.
#### Software:
- Have WordPress installed via an Ansible playbook.
- Have the database installed via an Ansible playbook.
- Test by going to the IP Address built in the Arch.
#### DevOps:
- Everything in Stage 2.
- Add some penetration testing for the RDS database.
- What are common techniques people use to get data, attempt these on the site to check it can withstand them.
- Add alerts so that I can be told when something is going wrong.
#### Just For Fun:
- If we get there, try and speed up the whole process of deployment.
- How fast can we make the deployment?
- Is it possible to do everything in under a minute?

### Stage 4 - EC2 (Hardcore) + RDS (Hardcore)
#### Arch:
- Everything in stage 3.
- Add an Amazon Machine Image representing the EC2 instance I would like to run.
- Automate the setup of the Amazon Machine Image somehow (Ansible?).
- Automate the deletion of Amazon Machine Images.
- Use the Amazon Machine Image in a load balancer setup that auto scales to match the load on the site.
#### Software:
- Figure out how to correctly load WordPress and other software bits and bobs into this setup.
- Once figured out, find a way of automating the process so it can be done at the click of a button.
#### DevOps:
- Everything in Stage 3.
- Simulate thousands of users accessing the WordPress site at any one time.
- Automate the gathering of data from AWS and show how that simulation affects site performance.
#### Just For Fun:
- If we get there, try and speed up the whole process of deployment.
- How fast can we make the deployment?
- Is it possible to do everything in under a minute?

---
# Building the Damn Thing
# 2024-10-02
## 22:34 - Working on the build
- I'm working on setting up the build now, just going to try and shit it out on AWS lickety split.
- Got a nice template created by `o1-preview`, going to try and see what happens when I put that into the application composer.

## 23:21 - Base template complete
- I've got a base template that I'm working with, it looks good in the application composer.
- I just need to run it now and see what happens.

## 23:32 - Setting up an AMI ahead of time
- I'm going to teach myself how to build an AMI from the terminal by running a CloudFormation script and then creating an AMI out of that.
- This should be interesting. This is the script I'm going to go with for now:
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: WordPress Dev Environment

Resources:
  WordPressInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      KeyName: your-key-pair-name
      ImageId: ami-0abc12345def67890 # Update with your preferred AMI ID
      SecurityGroupIds:
        - !Ref WordPressSecurityGroup
      UserData: !Base64
        Fn::Sub: |
          #!/bin/bash
          yum update -y
          yum install -y httpd php php-mysqlnd mariadb-server git
          systemctl start httpd
          systemctl enable httpd
          systemctl start mariadb
          systemctl enable mariadb
          git clone https://github.com/yourusername/your-repo.git /var/www/html/
          chown -R apache:apache /var/www/html/
          # Additional configuration commands
```
- I don't know if this will work, but let's see.

## 00:15 - Didn't make the AMI
- I haven't made the AMI yet because I'm caught up creating the GitHub repository for app.mumie.health.
- I've got it all set up and sent off now.