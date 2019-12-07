# kubernetes-devops
Implement a DevOps pipeline in a Kubernetes cluster running on AWS

A cluster was deployed on both Red Hat (Red Hat Enterprise Linux release 8.0 - Ootpa) and Ubuntu (Ubuntu 18.04.3 LTS) running on AWS. The setup and installation steps were similar for both clusters each consisting of the deployment VM (t2.micro), a master (c4.large), and 2 nodes (t2.medium)

Steps (based on the Red Hat instance, run as root user)

## 1. Launch the Instance
Under the EC2 Dashboard select "Red Hat Enterprise Linux 8 (64-bit x86)" as the launch instance.

Other parameters:
* Set a "Name" tag (i.e., k8s_cluster)
* Create and download new key pair (.pem format) or use an existing one
* Note the public DNS and ssh into the instance:
```
chmod 600 mykey.pem
ssh -i "mykey.pem" ec2-user@ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com
```
* Once logged in, run _sudo su_

## 2. Install AWS
```
 curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
 yum install unzip python2
 ln -s /bin/python2 /bin/python
 unzip awscli-bundle.zip
 ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
```
 RHEL: Ran _apt install unzip python_ (the "ln" step not needed)`

 Run "aws --version" to verify installation worked.
