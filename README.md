# kubernetes-devops
Implement a DevOps pipeline in a Kubernetes cluster running on AWS

A cluster was deployed on both Red Hat (Red Hat Enterprise Linux release 8.0 - Ootpa) and Ubuntu (Ubuntu 18.04.3 LTS) running on AWS. The setup and installation steps were similar for both clusters each consisting of the deployment VM (t2.micro), a master (m3.medium or c4.large), and 2 nodes (t2.medium)

Steps (based on the Red Hat instance, run as root user)

## 1. Launch the Instance
Under the EC2 Dashboard select "Red Hat Enterprise Linux 8 (64-bit x86)" as the launch instance.

Other parameters:
* Set a "Name" tag (i.e., k8s-cluster). We will call this the management instance.
* Create and download new key pair (.pem format) or use an existing one
* Note the public DNS and ssh into the instance:
```
chmod 600 mykey.pem
ssh -i "mykey.pem" ec2-user@ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com
```

* ssh into the instance and once logged in, run _sudo su_

## 2. Create and Attach a Security Group
* Click on the "EC2 Dashboard" -> "Security groups" -> "Create Security Group"
* Fill out the "Security group name" field (i.e., k8s-cluster) and "Description"
* Create/select the needed rules using "Add Rule"

* Go back to the "EC2 Dashboard" -> "Running instances"
* Select the "k8s-cluster" instance you just created
* Go to "Actions" -> "Networking" -> "Change Security Groups"
* Assign the newly created security group


## 3. Install AWS
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

## 4. Install Kubectl
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl 
mv ./kubectl /usr/local/bin/kubectl
```

## 5. Create an IAM User/Role with Full Access
* From the dashboard under "Services" -> "Security, Identity, & Compliance" select "IAM"
* Click on "Roles" (left menu) and "Create role"
* Select "EC2" as the AWS Service`
* Click on "Next: Permissions" and check "AmazonRoute53FullAccess", "AmazonEC2ContainerServiceFullAccess", "IAMFullAccess", and "AmazonS3FullAccess"
* Click the "Next: Tags" section and create a "Name" tag with value "k8s-role"
* Click on "Next: Review", ensure that all policies just added are there, and enter a "Role name" (i.e., "k8s-role"), click on "Create role"

## 6. Attached the Newly Created Role to the Server
* Select the instance on the EC2 Dashboard
* Go to "Actions" -> "Instance Settings" -> "Attach/Replace IAM Role"
* Select the newly created role and click "Apply"
* Run _aws configure_ leaving all parameters blank except the _Default region name_ which can be obtained from the instance details _Availability zone_.

## 7. Install _kops_, the Application User to Spin Up the Cluster, by Running Below Commands:
```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64 
sudo mv kops-linux-amd64 /usr/local/bin/kops
```
## 8. From the EC2 Dashboard Create a _Route53_ Private Hosted Zone
* Go to "Services" and search for "Route 53"
* Under "DNS Management" (section 1) click on "Create Hosted Zone"
* Fill the "Domain Name" field (i.e., mylastname.in)
* Select "Private Hosted Zone" (unless you have a registered Domain Name)
* For "VPC ID" select an id associated with your region
* Click on "Create"

## 9. Create and S3 Bucket
* Run _aws s3 mb s3://dev.k8s.mylastname.in_
* You can also confirm that bucket has been created by going to the dashboard "Services" and search for "S3"
* Edit the ~/.bashrc to include the variable _export KOPS_STATE_STORE=s3://dev.k8s.mylastname.in_ and run source ~/.bashrc

## 10. Create the Cluster Key
* Run _ssh-keygen_ to create the key (use default options and no passphrase)

## 11. Create the Cluster
* Create the cluster configuration on the S3 bucket by running:
```
kops create cluster --cloud=aws --zones=us-east-1c --name=dev.k8s.mylastname.in --dns-zone=mylastname.in --dns private
```
* Configure your cluster with: _kops update cluster --name dev.k8s.mylastname.in --yes_ (__Note:__ If you get an error message _UnauthorizedOperation: You are not authorized to perform this operation._ you should be able to resolve this error by creating a new role such as "k8s-role-admin" in step 5 and assign it blanket "AdministratorAccess")
* If the cluster was successful, you should see in your dashboard 4 running instances including the master and two nodes you just spun up:

![Alt text](/images/running_instances.png?raw=true "Running Instances")

## 12. Access the Cluster
* Log into the master node as _admin_ user (i.e., _ssh -i ~/.ssh/id_rsa admin@ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com_) from you management instance.
* Run a command like _kubectl get nodes_ to view the running nodes and add option _--show-labels_ for detailed configuration (you can alias the regularly used kubectl command as _alias k="kubectl"_ and add it to the admin ~/.bashrc)

## 13. __IMPORTANT__: Delete the Cluster
* If you are on a small budget I highly recommend you run _kops delete cluster dev.k8s.mylastnane.in --yes_ once you are done as AWS charges tend tyo skyrocket pretty fast when running a cluster (you can always easily re-create the cluster with your stored configuration). Note that simply stopping or even terminating the instances on the dashboard will not work!
__To re-create a deleted cluster repeat step #11__

## 14. Install Helm on the Master Node (to be used to spin up cluster applications)
* Log into the master node as _admin_ user (i.e., _ssh -i ~/.ssh/id_rsa admin@ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com_) from you management instance.
Run below commands:
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```



