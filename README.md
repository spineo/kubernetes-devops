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

 Run _aws --version_ to verify installation worked.
 
 Edit the ./bashrc to include the script path (i.e., add _export PATH=/usr/local/bin:$PATH_) and then run _source .bashrc_

## 3. Install Kubectl
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl 
mv ./kubectl /usr/local/bin/kubectl
```

## 4. Create an IAM User/Role with Full Access
* From the dashboard under "Services" -> "Security, Identity, & Compliance" select "IAM"
* Click on "Roles" (left menu) and "Create role"
* Select "EC2" as the AWS Service`
* Click on "Next: Permissions" and check "AmazonRoute53FullAccess", "AmazonEC2ContainerServiceFullAccess", "IAMFullAccess", and "AmazonS3FullAccess"
* Skip the "Next: Tags" section
* Click on "Next: Review", ensure that all policies just added are there, and enter a "Role name" (i.e., "k8s-role"), click on "Create role"

## 5. Attached the Newly Created Role to the Server
* Run _aws configure_ leaving all parameters blank except the _Default region name_ which can be obtained from the instance details _Availability zone_.

## 6. Install _kops_, the Application User to Spin Up the Cluster, by Running Below Commands:
```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64 
sudo mv kops-linux-amd64 /usr/local/bin/kops
```
## 7. From the EC2 Dashboard Create a _Route53_ Private Hosted Zone
* Go to "Services" and search for "Route 53"
* Under "DNS Management" (section 1) click on "Create Hosted Zone"
* Fill the "Domain Name" field (i.e., mylastname.in)
* Select "Private Hosted Zone" (unless you have a registered Domain Name)
* For "VPC ID" select an id associated with your region
* Click on "Create"

