## Package contents


1. Infra home assessment task.pdf - Test content
2. ansible_config - Ansible roles used in the test
3. CloudFormationTemplate.yml - CloudFormation template to initiate the required resources
4. README.md - This file

## Preface


The application is deployed in AWS cloud platform. I am using Red Hat Enterprise Linux 7.3 AMI (ami-40a8bf24). For the given task I decided to deploy three EC2 instances (t2.micro): RevProxy, AppServer1, AppServer2.

The sample "Hello World" application is a .war package originating from tomcat (see reference 5). It is deployed in two AppServers, each placed in different region (eu-west-2a and eu-west-2b) to provide the high availability.

The RevProxy server acts as a reverse proxy to AppServers and as a load balancer between these two.

For build and deploy automation I am using Ansible.

## Deployment instructions


1. In the CloudFormationTemplate.yml file, modify the properties section to provide the relevant values for:
	- OperatorEmail - this e-mail will receive alarm notifications (line 4)
	- KeyName - this key will be used to create the EC2 instances (line 9)
	- SecurityGroups - this group will be assigned to each resource, must allow for inbound traffic on ports 22 and 8080 (line 13)

2. Using AWS CloudFormation, create a new stack using the CloudFormationTemplate.yml template

3. Look in OperatorEmail's inbox for an alarming e-mail, then confirm the subscription

4. Open the hosts file located in ansible_config directory and provide IPv4 public address for [revproxy] and two [appserver]

5. Open the file ./ansible_config/roles/apache/vars/RedHat.yml and provide IPv4 addresses for two AppServers (lines 21,22)

6. Your local machine will be used to trigger the Ansible playbook. Make sure you have a public key which can be added to RevProxy and two AppServers. If no key is present under ~/.ssh/id_rsa.pub, generate a new one by running: `ssh-keygen -t rsa`

7. Copy this public key and append to /home/ec2-user/.ssh/authorized_keys file in RevProxy and two Appservers.

8. On your local machine navigate to ansible_config folder and run: ansible-playbook -i hosts site.yml; this will:
	- install and configure the apache reverse proxy in RevProxy
	- install and configure the tomcat server in Appserver1 and Appserver2
	- deploy the sample.war application in Appserver1 and Appserver2

9. Verify if the deployment was successful: on your local machine, open a web browser and type the RevProxy URL which proxies to the AppServers: http://<RevProxy_IP_address>:8080/sample

## Next possible steps to tune the deployment

It would be advisable to enable the scaling for both AppServers in case of an increased number of requests. I did not provide a working solution for this particular functionality but I do know how important it is and I have the idea how to implement it. For this scenario I would use the scaling-out approach, that is adding additional instances when necessary.

1. Once the AppServers are deployed a new Launch Configuration should be created. This is a template which will be used to create new instances in case a scaling event occurs. The template provides the instance type, AMI, key pair, security groups and device mappings.

2. Secondly, a Scaling Group should be created, which is a collection of instances sharing similar characteristics, in our case these are the AppServer instances. Within the Scaling Group, a Scaling Policies can be created which will determine under what conditions scaling-out occurs. One of the conditions may be the CPU utilization exceeding a specific threshold. This is how an alarm, created by the CloudFormation template, can be utilized.

3. Lastly, once the new instances are running, an AWS CodeDeploy could be used to configure the newly added instances (e.g. configure tomcat, deploy the .war application) and modify the apache settings in RevProxy server.


## References

1. https://aws.amazon.com/documentation/
2. https://github.com/ansible/ansible-examples
3. https://oracle-base.com/articles/linux/apache-tomcat-8-installation-on-linux
4. https://galaxy.ansible.com/geerlingguy/apache/
5. https://tomcat.apache.org/tomcat-6.0-doc/appdev/sample/
