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

