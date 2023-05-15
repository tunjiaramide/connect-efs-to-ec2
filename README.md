# Set up NFS Server on EC2 Servers

In this archictecture, we will set up two EC2 instances in two different AZs, and make them share same data from a single NFS server. We will be using Amazon Linux 2


## Archictecture Diagram
![alt text](https://adetunjiaramide.s3.amazonaws.com/images/aws/EC2_NFS.png)

### Create an IAM Role
Create an EC2 Role and attach the AmazonElasticFileSystemClientFullAccess policy

![alt text](https://adetunjiaramide.s3.amazonaws.com/images/aws/iam_role.png)


### Create a SG
Create an SG, add inbound rules
Add NFS with TCP 2049, select public in source
Add SSH with TCP 22, select public in source

![alt text](https://adetunjiaramide.s3.amazonaws.com/images/aws/sg.png)


### Launch the EFS Server

Move to EFS dashboard to create one.

We will be using default VPC, choose a name and select customize

![alt text](https://adetunjiaramide.s3.amazonaws.com/images/aws/nfs_two.png)

For testing purpose, we disable automatic backups, choose bursting in performance settings and disable encryption and choose next

Remove all available zones not in use, just use 1a and 1b, then remove the default security group, and add the SG created earlier

![alt text](https://adetunjiaramide.s3.amazonaws.com/images/aws/nfs_three.png)

Click next and click create

Wait till has fully provisioned, under EFS created, check Network tab to make sure Mount target state has finised creating

![alt text](https://adetunjiaramide.s3.amazonaws.com/images/aws/nfs_four.png)



### Launch EC2 instance

Launch an EC2 instance, name it Web_server_one, choose Amazon Linux 2

Select the subnet us-east-1a

Select SG created earlier

![alt text](https://adetunjiaramide.s3.amazonaws.com/images/aws/instance_one.png)

Attach the IAM role created earlier and click create

![alt text](https://adetunjiaramide.s3.amazonaws.com/images/aws/instance_three.png)

Launch another EC2 instance, name it Web_server_two and select subnet us-east-1b,

Follow same steps to create the second instance

![alt text](https://adetunjiaramide.s3.amazonaws.com/images/aws/instances.png)


### Configure NFS Server on Web server one

Go into Web_server_one and connect using EC2 instance connect

Run the following commands below: 

sudo mkdir -p /efs/content <!-- Create folder that you will mount the EFS -->
sudo yum -y install amazon-efs-utils <!-- Install packages for EFS for Amazon linux -->
sudo nano /etc/fstab <!-- We mount the file system everytime by default -->
file-system-id:/ /efs/content efs _netdev,tls,iam 0 0  <!-- Replace file-system-id with NFS ID created earlier -->

![alt text](https://adetunjiaramide.s3.amazonaws.com/images/aws/linux.png)

sudo mount /efs/content
df -k
cd /efs/wp-content

![alt text](https://adetunjiaramide.s3.amazonaws.com/images/aws/linux_two.png)

sudo touch testfile.txt <!-- It will create the file in the NFS -->

# Configure NFS Server on Web server two and view the file created

sudo yum -y install amazon-efs-utils
sudo mkdir -p /efs/content
sudo nano /etc/fstab
file-system-id:/ /efs/content efs _netdev,tls,iam 0 0 <!-- Replace file-system-id with NFS ID created earlier -->
sudo mount /efs/content
ls -la

![alt text](https://adetunjiaramide.s3.amazonaws.com/images/aws/linux_three.png)

Check to see if you see testfile.txt is available





