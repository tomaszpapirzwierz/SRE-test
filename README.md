## Package contents


1. Infra home assessment task.pdf - Test content
2. ansible_config - Ansible roles used in the test
3. CloudFormationTemplate.yml - CloudFormation template to initiate the required resources
4. README.md - This file

## Preface


The application will is deployed in AWS cloud platform. I am using Red Hat Enterprise Linux 7.3 AMI (ami-40a8bf24). For the given task I decided to deploy three EC2 instances (t2.micro)
	- RevProxy
	- AppServer1
	- AppServer2

The sample "Hello World" application is a .war package originating from tomcat (see reference 5). It is deployed in two AppServers, each placed in different region (eu-west-2a and eu-west-2b) to provide the high availability.

The RevProxy server acts as a reverse proxy to AppServers and as a load balancer between these two.

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
	- install and configure the apache reverse proxy in [revproxy]
	- install and configure the tomcat server in Appserver1 and Appserver2
	- deploy the sample.war application in Appserver1 and Appserver2

9. Verify if the deployment was successful: on your local machine, open a web browser and type the RevProxy URL which proxies to the AppServers: http://<RevProxy_IP_address>:8080/sample


## References

1. https://aws.amazon.com/documentation/
2. https://github.com/ansible/ansible-examples
3. https://oracle-base.com/articles/linux/apache-tomcat-8-installation-on-linux
4. https://galaxy.ansible.com/geerlingguy/apache/
5. https://tomcat.apache.org/tomcat-6.0-doc/appdev/sample/
