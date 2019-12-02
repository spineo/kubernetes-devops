# kubernetes-devops
Implement a DevOps pipeline in a Kubernetes cluster running on AWS

A cluster was deployed on both Ubuntu (Ubuntu 18.04.3 LTS) and Red Hat (Red Hat Enterprise Linux release 8.0 - Ootpa) running on AWS. The setup and installation steps were similar for both clusters each consisting of the deployment VM (t2.micro), a master (c4.large), and 2 nodes (t2.medium)

Steps (based on the Ubuntu instance, run as root user)

## 1. Install AWS
```
 curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
 apt install unzip python
 unzip awscli-bundle.zip
 ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
```
 RHEL: Ran "yum install unzip python2" and created a symlink "ln -s /bin/python2 /bin/python"

 Run "aws --version" to verify installation worked.

