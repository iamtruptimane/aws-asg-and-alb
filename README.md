
# Working with Amazon EC2 Auto Scaling Groups and Application Load Balancer

In this PROJECT, We will learn how to create and configure all the resources that are needed to automatically scale an application in response to load.



## Learning Objectives
* Create and configure an Application Load Balancer
* Create and configure Auto Scaling groups and launch templates
* Utilize Auto Scaling to ensure application availability

## Prerequisites
You should be familiar with:

    * EC2 Basics including launching instances and connecting to them using SSH
    * Working with the AWS Console
    * Conceptual understanding of CloudWatch, and EC2 Security Groups (firewall rules) 

## Step 1: Logging In to the Amazon Web Services Console

## Step 2: Creating a Load Balancer using Elastic Load Balancing
1. In the AWS Management Console, in the search bar at the top, enter EC2, and under Services, click the EC2 result

2. In the left-hand menu, under Load Balancing, click Load Balancers

3. To start creating your load balancer, click Create Load Balancer

4. In the Application Load Balancer tile, click Create

5. In the Basic configuration section of the form, in the Load balancer name field, enter web

Ensure Scheme and IP address type are set to their defaults Internet-facing and IPv4.

6. In the Network mapping section, select all Mappings except us-west-2d:

7. In the Security groups section, deselect the default security group, and click the Create a new security group link:

8. In the Basic details section, enter the following:

    ```
    Security group name: Enter web
    Description: Enter ELB for a webserver cluster
    ```
9. In the Inbound rules section, click Add rule

10. Configure the new security group rule with the following options:
    ```
    Type: Select HTTP
    Source: Select Anywhere-IPv4
    ```

11. Scroll to the bottom and click Create security group

12. Return to your browser tab with the Create Application Load Balancer form open.

13. Click the refresh icon next to the Security groups drop-down and then select the security group you created

14. Unselect the default security group.

15. In the Listeners and routing section, click the Create target group link
    ```
    A listener defines which ports the load balancer will listen on, and which ports it will direct traffic to when balancing traffic across EC2 instances. The default of port 80 (HTTP) is what you will use in this lab.

    ```
16. In the Target group name textbox, enter web.

17. Scroll to the bottom of the page and click Next.
    ```
    The Register targets step will load. You do not need to register any targets as you will be using an auto scaling group to create instances later in the lab.

    ```
18. Scroll to the bottom and click Create target group.

19. Return to your browser tab with the Create Application Load Balancer form open.

20. Click the refresh icon next to the Default action drop-down and then select the target group you just created:

21. Scroll to the Summary section and review your load balancer's configuration.

22. Scroll to the bottom and click Create load balancer.

23. To return to the load balancer list page, click View load balancers.

In this step, we created an Elastic Load Balancer (ELB) to service HTTP requests on port 80 from any IP source address. This ELB will be used on the front end to direct requests to several instances running a web server. This is a very common use case for ELBs.

## Step 3: Creating a Launch Template
A Launch Template is a template that an Auto Scaling group uses to launch Amazon EC2 instances.

1. Navigate to EC2 in the AWS manegment cansole.

2. In the left-hand menu, under Network & Security, click Security Groups.

3. To start creating a new security group, click Create security group.

4. Under Basic details, in the Security group name field, enter webserver-cluster.

5. In the Description field, enter Webserver security group.

6. In the Inbound rules section, click Add rule.

7. To configure a rule allowing SSH traffic, enter the following values:

    ```
    Type: Enter SSH and select SSH
    Source: Select Anywhere-IPv4
    Description: Enter SSH
    ```
You have added this rule so that later you can access instances using SSH.

8. To add another inbound rule, click Add rule again in the Inbound rules section.

9. To configure a rule allowing HTTP traffic, enter the following values:
    ```
    Type: Enter HTTP and select HTTP
    Source: Select Anywhere-IPv4
    Description: Enter HTTP
    ```
10. To finish creating your security group, scroll to the bottom of the page and click Create security group:

11. In the left-hand menu, under Instances, click Launch Templates:

12. To open the Launch Templates page and click the Create launch template button:

13. In the Launch template name and description section, enter the following values accepting the defaults for fields not specified:

    ```
    Launch template name: webserver-cluster
    Template version description: Lab launch template
    Provide guidance to help me set up a template that I can use with EC2 Auto Scaling: checked 
    ```
14. Scroll down to the  Application and OS Images (Amazon machine Image) section, select the Amazon Linux box, and select the Amazon Linux 2 AMI (HVM) option.

15. In the Instance type field, enter t2.micro and click the t2.micro result.

16. In the Key pair (login) section, under Key pair name, select key-pair.

17. Under the Network Settings section, click the Security groups drop-down and select webserver-cluster.

18. Take a look at the Storage (volumes) section.

19. Scroll down to the bottom and click Advanced details.

20. Scroll down to the Detailed CloudWatch monitoring option, and select Enable.

21. In the User data text-box at the bottom of the page, enter the following script:

    ```
    #!/bin/bash
    # Enable the epel-release
    sudo amazon-linux-extras install epel
    # Install and start Apache web server
    sudo yum install -y httpd php
    # Start the httpd service
    service httpd start
    # Install CPU stress test tool
    sudo yum install -y stress
    ```

This bash script installs PHP, an Apache webserver (httpd), and a tool for stress testing called Stress.

Warning: The EC2 instances will never reach 100% CPU Utilization due to the limitations of the burstable credit. They should reach an usage of about 80%.

22. To create your Launch Template, click Create launch template.

24. To return to the EC2 management console, click View launch templates.

we have created a security group and we have created a launch template that can be used by an auto-scaling group to launch identical instances every time.

## Step 4: Creating an Auto Scaling Group
An Auto Scaling group is a representation of multiple Amazon EC2 instances that share similar characteristics and that are treated as a logical grouping for the purposes of instance scalling and management.

1. In the left-hand menu, under Auto Scaling, click Auto Scaling Groups.

2. To begin creating your auto-scaling group, click Create Auto Scaling group.

3. In the Name field, enter webserver-cluster.

4. In the Launch template field, select webserver-cluster.

5. To advance to the next page of the wizard, click Next.

6. In the Network section, click the Subnets drop-down and select each subnet except us-west-2d.

7. Click Next to move the next page of the wizard.

8. In the Load balancing - optional section, select Attach to an existing load balancer.

9. In the Existing load balancer target groups drop-down, select web | HTTP.

This is the target group you created when you created an application load balancer in a previous step.

10. In the Health checks section, enter the following values:

    ```
    Turn on Elastic Load Balancing health checks: checked
    Health check grace period: 120
    ```
11. In the Additional settings section, under Monitoring, check Enable group metrics collection within CloudWatch.

12. To proceed to the next step of the form, click Next.

13. In the Group size - optional section of the form, in the Maximum capacity text-box, enter 5.

Leave the other fields at their defaults, a Minimum and Desired capacity of one will mean that the auto-scaling group will attempt to have at least one healthy instance running at all times.

Specifying a desired capacity higher than the minimum will result in the auto-scaling group initially launching the desired amount of instances. If the instances are using less than the metric you specify in the scaling policy the auto-scaling group will reduce the number of instances until it reaches the minimum. Usually desired and minimum are set to the same value, but setting desired higher can be useful if you are expecting significant initial traffic temporarily and you don't want to have more instances running than you need for cost reasons.

14. In the Scaling policies - optional section, select Target tracking scaling policy, and enter the following values:

    ```
    Scaling policy name: Lab scaling policy
    Metric type: Select Average CPU utilization
    Target value: 80
    Instances warmup: 0
    Disable scale in to create only a scale-out policy: unchecked
    ```
15. To move to the next page of the wizard, click Next.

16. To advance to the next page of the form, click Next.

17. To move to the review section of the wizard, click Next.

18. To finish creating your auto-scaling group, at the bottom, click Create Auto Scaling group:

19. In the left-hand menu, under INSTANCES, click Instances.

This instance has been launched by your auto-scaling group because you specified the minimum capacity of one.

20. In the left-hand menu, click Auto Scaling Groups.

21. In the list of auto-scaling groups, click webserver-cluster.

22. To see the recently launched instance again, click the Instance management tab.

23. Navigate to load balencer in the EC2 cansole.

24. In the list of load balancers, select Web.

25. In the Details tab, copy the value of the DNS name field, and paste it into a new browser tab.

You will see the Apache test page.

This confirms that the Apache web server is installed on the instance and that your load balancer is routing traffic to the instance in your auto-scaling group.

In this step, we used the EC2 management console to create an auto-scaling group, and we tested that our load balancer routes traffic to our auto-scaling group.

## Step 5: Testing the Auto Scaling Group

1. In the left-hand menu, under Instances, click Instances:

2. To connect to the running instance, select it in the table and click Connect:

3. Ensure the EC2 Instance Connect tab is selected and at the bottom, click Connect.

4. To verify that the stress testing tool is installed, in the command line of the instance, enter the following command:

    ```
    which stress
    ```
You will see the following in response:

    ```
    \usr\bin\stress

    ```
5. To start a stress test that will increase CPU utilization, enter the following command:

    ```
    stress --cpu 2 --io 1 --vm 1 --vm-bytes 128M --timeout 5m

    ```

This command starts a stress test that will last five minutes and does the following:

    ```
    Runs two workers that will use a large amount of CPU
    Runs one worker that will use a large amount of IO (input/output)
    Runs one worker that will use one-hundred and twenty-eight megabytes of memory

    ```
6. Navigate to instances.

7. Select the running instance.

8. In the tabs under the list of instances, click Monitoring:

9. To see the CPU utilization of the instance in more detail, in the top-right of the CPU utilization card, click the three dots, and click View in metrics:

10. To change the graph resolution, under Period, select 1 minute:

11. To see data from the last 15 minutes, at the top, click custom, and click 15 in the Minutes row:

12. Return to your browser tab with EC2 instances open.

13. Navigate to auto scaling grup in the EC2 management console.


14. In the list of auto-scaling groups, click webserver-cluster.

15. To see scaling events for our auto-scaling group, click the Activity tab.

In this step, we connected to the EC2 instance and we executed a stress test. we also used the EC2 management console to observe our auto-scaling group responding to the stress test.

In this step, wehave created a Classic load balancer, a launch template, an auto-scaling group, and finally, we saw our auto-scaling group scale in response to high CPU utilization.
















