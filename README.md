# Uninterrupted Evolution: Blue-Green Deployment with AWS ALB for Seamless Application Updates
In this section, we will learn how to deploy new releases/updates of your application without affecting the performance by ensuring high availability and reliability. We will leverage the features of the Application Load balancer to implement blue/green deployment. <br/>

A Blue/Green deployment is a deployment strategy in which you create two separate, but identical environments. One environment (blue) is running the current application version and one environment (green) is running the new application version. Using a blue/green deployment strategy increases application availability and reduces deployment risk by simplifying the rollback process if a deployment fails. Once testing has been completed on the green environment, live application traffic is directed to the green environment, and the blue environment is deprecated.

In the below image, we have two production environments, Blue and Green. The Blue environment is one where the current (let’s say version 1) of the application is running and is live as well. All the traffic of the application is directed to Blue by the Router or load balancer (as per the infra setup). Meanwhile, version 2 of the application is deployed and tested on Green Environment.

![image](https://github.com/user-attachments/assets/53429640-5762-481c-81a5-64c748b8096c)

Currently, at this stage, we can refer to the Blue environment as Live and Green as idle or at staging state.

Once the code for version 2 is tested and ready to be live for production, we redirect the traffic from the Blue Environment to the Green Environment, making Green Live and Blue a staging environment. If any problem is detected with the Infrastructure or application after version 2 is made live, we can roll back to the previous version just by swapping the environment.

![image](https://github.com/user-attachments/assets/1f85cec5-6ece-4e11-9db6-1308ff4ca3ab)

Let's see how this can be done.

### 1. Deploying App Version 1.
  - Create Launch template (shopping-app-version-1)
  - Create an Auto Scaling Group (shopping-app-version1-asg)
  - Create a Target Group (app-version1-tg)
  - Attach Auto Scaling Group (shopping-app-version1-asg) to Target Group (app-version1-tg)
  - Create Application Load Balancer (shopping)
  - Forwarding traffic 100% to the target group (app-version1-tg) using ALB rules.

### 2. Deploying App Version 2. <br/>
Rolling out the App’s New version. Application with new features has to be deployed in other target groups.

  - Create Launch configuration (app-version2-lc).
  - Create Auto Scaling Group (app-version2-asg).
  - Create a Target group (app-version2-tg).
  - Attach Auto Scaling Group (app-version2-asg) to Target Group (app-version2-tg).
  - Forwarding 100% of traffic to a new target group (app-version2-tg) using ALB rules.

### 3. Enabling group level stickiness

## Glossary

Application Load balancer is used to route the requests to a specific target group(servers) based on the rules defined in it.

The Target group is a group of servers/instances that process and serve the request back to the Application Load balancer

The auto-scaling group is used to automatically provision new instances so as to handle the peak requests. To what limit the Autoscaling group could create new instances is defined in min, max, and desired capacity. By using the scaling policies of the Autoscaling group, we can set to provision new instances based on resource utilization metrics (dynamic scaling policies).ASG is attached to the target group so that the new instances created by the Autoscaling group would be attached to the corresponding target group. So, the application load balancer would start to send requests to the new instance which is recently attached to the target group by the Autoscaling group

Launch configuration is a template that contains the configuration and specification of an instance which the Autoscaling group uses to provision new instances

Let’s first look at the deployment of version1 of the App under an Application Load balancer.

## 1. Deploying App Version 1

## Step 1: Create Launch configuration (shopping-app-version-1)

Login to AWS console >> EC2 >> Launch Template >> Create launch template

![lc](https://github.com/user-attachments/assets/bad0d112-7c4b-4013-9e2f-bad2158509fa)

**Launch configuration Name**: shopping-app-version-1

**AMI**: ami-0xxxxxxx (You can get Amazon Linux ID from EC2 Instance Launch Panel)

**Instance type**: t2.micro

**Advanced configuration** >> Advanced details >> Userdata: Enter the set of commands that has to be executed automatically after the instance is provisioned

If your application is pre-packaged with a basic image, then you might choose that particular AMI. Otherwise, you can make use of user data to deploy the application.

![image](https://github.com/user-attachments/assets/70eeb2e3-9736-4430-99de-cd56d67c65d8)

![image](https://github.com/user-attachments/assets/488105f7-4f9a-45ad-9b96-a8d8008b0578)

![image](https://github.com/user-attachments/assets/81587009-00d0-4fb2-886a-7f8ad732bed7)

**Storage(volumes)**: By default, 8GB Root volume is attached for t2.micro class

![image](https://github.com/user-attachments/assets/8143cb27-9651-42e4-9b1c-96033d5d7c43)

**Security groups**: Select from existing security groups. If you don’t have any existing security groups, you can create a new one using “create a new security group”

**Key pair** >> Choose an existing key pair

![image](https://github.com/user-attachments/assets/e4361042-1a89-4a67-88f4-d74d65a9242f)

## Step 2: Create Auto Scaling Group (shopping-app-version1-asg)

To create AutoScalingGroup,

In the EC2 console, click on EC2 >> Auto Scaling groups >> Create Auto Scaling group

Name : shopping-app-version1-asg

Click on Launch Template

Select the launch template in the drop-down as “shopping-app-version1” >> click Next

![image](https://github.com/user-attachments/assets/40cd0b71-ac65-44c4-b8cb-261fd055ba12)

Select the default VPC

Availability Zones and subnets: Select the subnets in which ASG can create the instances

![image](https://github.com/user-attachments/assets/c8df3f78-5f6c-4942-92a2-107f437a368a)

Load Balancer is not selected now 

Health check grace period : This time period delays the first health check until your instances finish initializing. It doesn't prevent an instance from terminating when placed into a non-running state.

120 Sec

![image](https://github.com/user-attachments/assets/d8dbae19-5aa4-47f7-b0c7-129989f01a50)

Configure group size and scaling policies

Min, Max, Desired capacity — 2

Scaling policies

Automatic scaling policy — dynamically resize your Auto Scaling group to meet changes in demand

As of now, choosing None to use a static scaling policy

![image](https://github.com/user-attachments/assets/6f3f55ee-80be-4ef6-a399-8058da92f7e7)

Add tags

Enter the name and value as tags to add to the new instances that the AutoScalingGroup creates so that we can understand the instances that are created by the AutoScalingGroup

Key: Name, Value: version1

![image](https://github.com/user-attachments/assets/8a4b182d-2b0d-4fb4-a00b-53f8acfe1722)

Review

On this page, we can review the Auto-scaling group configuration and if any section that needs to be modified, click on the edit option on the same section to make changes

Click on Create Auto Scaling Group

The AutoScalingGroup is created.

The details section explains the capacity details of AutoScalingGroup

The instance management section shows the instances that are managed by the AutoScalingGroup.

## Step 3: Create Target group (app-version1-tg)

A Target group tells a load balancer where to direct traffic to: EC2 instances, fixed IP addresses; or AWS Lambda functions, amongst others.

EC2 management console >> Click on Target Group under Load Balancing >> Click on Create Target Group

Choose a target type: Instances

![image](https://github.com/user-attachments/assets/c17cbb83-b430-4612-b7ea-e0f446952bfc)

**Target group name**: app-version1-tg

**Protocol**: HTTP
**Port**: By default, a load balancer routes requests to its targets using the protocol and port number that you specified when you created the target group.
Instance Port: 80


![image](https://github.com/user-attachments/assets/8874487c-6f28-419a-8b8c-8893ac30e685)

**Health checks**: The associated load balancer periodically sends requests, per the settings provided, to the registered targets to test their status

**Health check protocol**: HTTP
**Health check path**: /index.php

Advanced health check settings

**Healthy threshold**: 2 (It is the number of consecutive health checks successes required before considering an unhealthy target healthy)

**Unhealthy threshold**: 2 (It is the number of consecutive health check failures required before considering a target unhealthy)

**Timeout**: 5 seconds (It is the amount of time, in seconds, during which no response means a failed health check)

**Interval**: 30 seconds (It is the approximate amount of time between health checks of an individual target)

**Success codes**: 200


![image](https://github.com/user-attachments/assets/0e0cdf4d-907d-4241-a3c4-bbf027553d40)

**Register targets**: Do not select any specific instance as targets. Why registering instances as targets is not recommended here is, that we will be attaching an autoscaling group to the target group. If we select and register a particular instance from the list shown in the pic, when that particular instance is down ASG will create a new instance following the health check failed situation. But the new instance which is replaced by the ASG would need to be manually registered to the target group. Hence, We will attach an auto-scaling group to the target group so that the instances created by the ASG will automatically attached as targets for the load balancer.

Click on Create Target Group. A Target Group with the name “app-version1-tg” will be created.


![image](https://github.com/user-attachments/assets/0ba04d03-1e3a-43c4-a161-506de126c7ea)

We can see the Target Group with the name “app-version1-tg” is created with no registered targets. In the next section, we will register the targets that are created by the auto-scaling group indirectly by attaching the auto-scaling group to the target group


![image](https://github.com/user-attachments/assets/bc4c513c-3ec4-4dec-96e2-499a340146aa)

## Step 4: Attach Auto Scaling Group to Target Group.

EC2 management console >> Auto scaling Group >> Select the Auto scaling Group “shopping-app-version1-asg” >> Edit

Under the Load balancing option, select “Application, Network or Gateway Load Balancer target groups” select the Target group we created “app-version1-tg” and click on update.

![image](https://github.com/user-attachments/assets/376de532-c28f-48da-ab25-bdfd257b14a4)

Now the Instances created by the Auto scaling Group are added to the Target Group.


![image](https://github.com/user-attachments/assets/dd41c0b1-f7d2-4434-a9b5-b326674cf5a0)

## Step 5: Create Application Load balancer (shopping)

In the EC2 management console, Click on Load Balancer >> Create Load Balancer >> Application Load Balancer >> Create

**Load balancer name**: shopping
**Network mapping**: Select at least two Availability Zones and one subnet per zone. The load balancer routes traffic to targets in these Availability Zones only.


![image](https://github.com/user-attachments/assets/a2098e89-36a7-401f-ab63-5372e1752c7b)

**Security Group** : Select an existing security group : LB-SG/Mumbai-region

**Listeners and routing** :

Application load balancer endpoint would listen only on this particular listener protocol and port number. Application load balancer won’t support any other protocol and port other than what is defined as a listener.

If a request comes with https protocol for the 443 port, the default Action we select is Forward to the Target Group we created app-version1-tg which means HTTP requests that come to the load balancer, would be forwarded to the underlying target group.


![image](https://github.com/user-attachments/assets/a32749fb-c098-4e07-8a6f-53caf7fa52c9)

## Using AWS Certificate Manager (ACM) we can provision and manage SSL/TLS certificates for your AWS-based websites and applications.

Since ALB is configured to listen on HTTPS port only, We are loading the certificate created for jeethu.shop domain from ACM. This certificate will automatically be added to the listener certificate list.


![image](https://github.com/user-attachments/assets/804213d2-375b-45e1-84f4-166e88e91a3e)

Now Click on Create Load Balancer.

## Step 5(1): Add HTTP listener to redirect to HTTPS

Another listener for HTTP(80) is added to ensure the requests that come with HTTP protocol are also served properly.

## EC2 >> Load Balancer >> Select the load balancer “shopping” >> Under Listeners tab >> Add Listener

Protocol: HTTP
Port: 80
Default action: Redirect to HTTPS: 443
Click on Add


![image](https://github.com/user-attachments/assets/90bd5e5f-6c82-49dc-8d10-fb2b376833f3)

## Step 6: Forwarding traffic to target group 1 (app-version1-tg) using ALB rules

### EC2 >> Load Balancer >> Select the load balancer “shopping” >> Under Listeners tab >> Select the Listener “HTTPS:443” >> Actions >> Manage rules


![image](https://github.com/user-attachments/assets/79f9eabb-62d3-446b-9438-b56ac7a4d6a9)

The requests that come with the shopping.jethu.shop host header would be subjected to the first rule which forwards the request to the target group containing the version instances.

All other requests (not containing shopping.jeethu.shop host header) would be subjected to the “default action” rule(second rule in the pic). By default, it is also set to redirect to app-version1-tg. We will edit it to return fixed responses as “Site not found”.


![image](https://github.com/user-attachments/assets/26fa4758-7d79-4a67-824a-3806127dbaf2)


Click on Add action >> Return Fixed Response

Response code: 503
Content-Type: text/HTML
Response body: Site Not Found


![image](https://github.com/user-attachments/assets/d3d5eada-89ad-4852-8d2e-2c23ec264caf)

The Default rule was successfully updated.

## Step 7: Pointing my domain “shopping.jeethu.shop” to the ALB endpoint in route 53

Navigate to Route 53 Dashboard >> Click on Hosted Zone


![image](https://github.com/user-attachments/assets/248c98de-dedb-4f35-b462-5da3d7ab3ffc)

Record name: shopping
Enable: Alias
Route traffic to Alias to Application and Classic Load Balancer
Choose Region: Asia Pacific Mumbai [ap-south-1]
Choose Load Balancer: dualstack.Shopping-app-xxxxx.ap-south-1.elb.amazonaws.com
Click on Create.

Now calling https://shopping.jeethu.shop  works fine.

## 2. Deploying App Version 2

### Step 8: Create Launch template (shopping-app-version2)

Launch Configuration Name: shopping-app-version2

AMI: ami-0xxxxxxx (You can get Amazon Linux ID from EC2 Instance Launch Panel)

Instance type: t2.micro

Advanced configuration >> Advanced details >> Userdata: Enter the set of commands that has to be executed automatically after the instance is provisioned


![image](https://github.com/user-attachments/assets/6f1b5b2b-0e5a-4cca-823e-8fa1b9e7328c)

We can see 2 launch configurations, One is for version 1 and another is for version 2.

## Step 9: Create Auto Scaling Group (shopping-app-version2)

To create AutoScalingGroup,

EC2 >> Auto Scaling groups >> Create Auto Scaling group

Name : shopping-app-version2

Click on Launch Template

Select the launch template in the drop-down as “shopping-app-version2” >> click Next

Add tags

Enter the name and value as tags to add to the new instances that the AutoScalingGroup creates so that we can understand the instances that are created by the AutoScalingGroup

Key: Name, Value: version2

Click on “Create Auto Scaling Group”

Now we can see the AutoScalingGroup is created.

The Details section explains the capacity details of AutoScalingGroup

The instance management section shows the instances that are managed by the Auto Scaling Group

## Step 10: Create Target group (app-version2-tg)

A Target group tells a load balancer where to direct traffic to: EC2 instances, fixed IP addresses; or AWS Lambda functions, amongst others.

Target group name: app-version2-tg Protocol: HTTP Port: By default, a load balancer routes requests to its targets using the protocol and port number that you specified when you created the target group. Instance Port: 80

Health checks: The associated load balancer periodically sends requests, per the settings provided, to the registered targets to test their status

Health check protocol: HTTP Health check path: /index.php

Register targets: Do not select any specific instance as targets. Why registering instances as targets is not recommended is, that when that particular instance is down ASG will create a new instance following the health check failed situation. But the new instance which is replaced by the ASG would need to be manually registered to the target group. Hence, We will attach an auto-scaling group to the target group so that the instances created by the ASG will automatically act as targets for the load balancer.

Click on Create Target Group. A Target Group with the name “app-version2-tg” will be created.

Select the target group “app-version2-tg” from the list, you can see no targets under “Registered targets.”

## Step 11: Attach Auto Scaling Group to Target Group

EC2 management console >> Auto-scaling Group >> Select the Auto-scaling Group “app-version2-asg” >> Edit.

Under the Load balancing option, select “Application, Network or Gateway the Load Balancer target groups” Select the Target group we created “app-version2-tg” and click on update.

The instances of the auto-scaling group are attached to the target group now.

## Step 12: Forward 100% of traffic to the new target group (app-version2-tg) using ALB rules.

EC2 console >> Load Balancers >> Click on shopping

Under Listener tab >> Choose HTTPS:443 listener >> Actions >> Manage rules

Edit the rule number 1 where IF = Host is shopping.jeethu.shop
Select a second target group from the dropdown >> Choose app-version2-tg

![image](https://github.com/user-attachments/assets/66191a24-7334-4db5-9af1-d1bdca443f61)

Enter the weightage in the small box next to the target group box app-version1-tg: 0 (0%) app-version2-tg: 1 (100%)

This allows the ALB to forward 100% of the traffic to the app-version2-tg target group and No traffic to the app-version1-tg target group.

The website/application shopping.jeethu.sop is served fully by the app-version2-tg target group and not by the app-version1-tg target group as in the screenshot

![image](https://github.com/user-attachments/assets/e70449ce-3d79-4d3c-a4d4-ab878883f846)
