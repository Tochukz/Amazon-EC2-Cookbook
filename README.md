# Amazon EC2 Cookbook  (2015)
__By Sekhar Reddy and Autobindo Sarkar__  

## Chapter 1: Selecting and Configuring Amazon EC2 Instances  
__Choosing the right AWS EC2 instance types__  
If you require higher storage performance, then ensure that the EC2 instance type you choose supports SSD.  
There are three distinct purchasing options available for provisioning the AWS EC2 instances:  
* __On-demand instances__: These instances are billed on an hourly basis and no upfront payment are required. This is the default purchasing option in AWS. It is suitable for unpredictable workloads or short-duration requirements.  
* __Spot instances__: The provisioning is done through a bidding process. There are no upfront costs. It is cheaper than the on-demand instances and suited for low compute applications.  
* __Reserved instances__: They are about 50-60% cheaper than on-demand instances. Available for 1-3 year plans and suited for workload that require instances for long duration.

See [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/) for descriptions and typical use cases for each EC2 instance type.  

On AWS, there are two types of tenancy, _dedicated_ and _shared_. Tenancy can be configured at the instance level or at the VPC level. Once the option is selected, changing the tenancy type (instance or VPC level) is not allowed.    

Amazon EBS-optimized instances deliver dedicated throughput to Amazon EBS, with options ranging between 500 Mbps and 2,000 Mbps ( depending on the instance type selected). EBS OptimizedEC2 instances also allocated dedicated bandwidth to its attached volumes.  

When we use an EBS-backed instance, we have the option of using either the instance's storage device or the root device which is the EBS instance. The instance size may be changes subsequently or stopped to stop billing but any data store in the instance's storage will be lost.  The associated EBS instance on the other hand cannot have it's size changed or stopped but can only be terminated.

__Getting access key ID and secret access key__  
Instead of generating these credentials from the root account, it's always best practice to use IAM users. To generate AWS credential:  
1. Login to the AWS management console.
2. Click on your account name and then click __My Security Credentials__  
3. Click on the `Create Access Key` button to generate your access key.  
4. Download the `.csv` key file and keep it in a safe place.

See page[22] for instruction on how to install AWS CLI on Linux, Windows and Mac.

After installing AWS CLI try the command:  
```
$ aws ec2 describe-regions  
```
This will list the regions available in AWS.

__Launching EC2 instances using EC2-Classic and EC2-VPC__  
If you attach an EIP (Elastic IP) to EC2-Classic instance, it will get dissociated when you stop the instance. But for VPC EC2 instance, it remains associated even after you stop it. We can create subnets, routing tables, and Internet gateways in VPC.  

1. First, you create a key pair
```
$ aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem
```  
Set permission of your private key file so that only you can read it
```
$ chmod 400 MyKeyPair.pem
```
To verify that the private key matches the public key stored in AWS, you can display the fingerprint for your keypair  
```
$ aws ec2 describe-key-pairs --key-name MyKeyPair
```  
To list all your key pairs
```
$  aws ec2 describe-key-pairs
```
To delete you keypair in the future  
```
$ aws ec2 delete-key-pair --key-name MyKeyPair
```  
2. Create a security group  
A security group operates as a firewall with rules that determine what network traffic to allow. The security group can be used in a VPC or EC2-Classic shared flat network.   
To create a security group for a specific VPC  
```
$ aws ec2 create-security-group --group-name my-sg --description "My security group" --vpc-id vpc-1a2b3c4d
```
This will generate a GoupId such as `sg-903004f8`
To view details about the security group, use the `group-ids` flag
```
$ aws ec2 describe-security-groups --group-ids sg-903004f8
```
To create a security group for EC2-Classic
```
$ aws ec2 create-security-group --group-name my-sg --description "My security group"
```
To view details about the security group you can use the `group-ids` or `group-name` flag   
```
$ aws ec2 describe-security-groups --group-names my-sg
```
To delete a security group  
```
$ aws ec2 delete-security-group --group-id sg-xxxxxxxxxxx
```
3. Add rules to your security group  
To add a rule to the security group for EC2-VPC, use the `group-id` of the security group.
For windows instance, add a rule to allow inbound traffic on TCP 3389 to support Remote Desktop Protocol.  
```
$ aws ec2 authorize-security-group-ingress --group-id sg-903004f8 --protocol tcp --port 3389 --cidr 203.0.113.0/24
```
For linux instance, add a rule to  inbound traffic on TCP port 22 to support SSH connections.
```
$ aws ec2 authorize-security-group-ingress --group-id sg-903004f8 --protocol tcp --port 22 --cidr 203.0.113.0/24
```
Note the `203.0.113.0/24` represents you public IP and range.
To view the changes to the security group, you can run the `describe-security-groups` command again.   
To remove a rule from your security group  
```
$ aws ec2 revoke-security-group-ingress --group-id sg-903004f8 --protocol tcp --port 22 --cidr 203.0.113.0/24
```
To add a rule to the security group for EC2-Classic you may either use the `group-id` or the `group-name`.  
 ```
 $ aws ec2 authorize-security-group-ingress --group-name my-sg --protocol tcp --port 22 --cidr 203.0.113.0/24
 ```  
To delete a security group  
```
$ aws ec2 delete-security-group --group-id sg-903004f8
```
This will not work if the security group is currently attached to an environment. For EC2-Classic you can also use the `group-name` instead of the `group-id`

4. Get an AMI ID.
Go to EC2 Management console and click on the `AMI` link under __Images__ on the left navigation bar. Copy the image ID of your choice image from the list of AMIs. For _Ubuntu Server 20.04 LTS (HVM), SSD_ the ID is `ami-096cb92bb3580c759` and for _Microsoft Windows Server 2019 Base_ it is `ami-08698c6c1186276cc`.
5. Lunch your instance using the AMI ID an
```
$ aws ec2 run-instances --image-id ami-0244a5621d426859b --count 1 --instance-type t2.micro --key-name MyKeyPair --security-group-ids sg-903004f8
```  
For EC2-Classic, you can use the `--security-groups` flag (with the group name as value) instead of the `--security-group-ids` flag which must be used for EC2-VPC.  

6. To stop the instance
First display the instance details:
```
$ aws ec2 describe-instances
```
Copy the `InstanceId` from the details. Then use the `InstanceId` to stop the instance.  
```
$ aws ec2 stop-instances --instance-ids i-xxxxxxxxxxx
```

7. To start the instance again
```
$ aws ec2 start-instances --instance-ids i-xxxxxxxxxxx
```

8. Connect to the instance via SSH  
The instance must be running, so view instance details
```
$ aws ec2 describe-instances
```  
Then copy the instance's `PublicDnsName`. It will be used  for the `ssh` command.  
Make sure your local computer's public IP is listed in the instance security group `IpPermissions`.
```
$ aws ec2 describe-security-groups
```
If you public IP is not listed, then add it to the security group
```
$ aws ec2 authorize-security-group-ingress --group-id sg-903004f8 --protocol tcp --port 3389 --cidr 203.0.113.0/24
```
Where you public IP is 203.0.113.0.  
SSH using the DNS name and your key pair
```
$ ssh -i your-key-pair.pem your-instance-user@your-instance-public-dns-name
```
__NB:__ The AMI ID for a given OS varies from one region to another. So to avoid _InvalidAMIID.NotFound_ error, make sure that the active region on your console when you copy the AMI ID is the same as the default region set in you `~/.aws/config` file.   
You can override the default region set in you config file by using the region flag `--region eu-west-2` or setting the `AWS_REGION` environment variable on your terminal window.  
9. To terminate the instance  
```
$ aws ec2 terminate-instances --instance-id i-xxxxxxxx
```
10. To add or replace key pair for already existing instances
Generate a private key
```
$ aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > my-key-pair.pem
```
Retrieve the public key from the private key
```
$ ssh-keygen -y -f /path_to_aws key_pair/my-key-pair.pem
```
If you get an error you can generate a new private key using the management console (EC2 -> Key Pairs) and then try this command again.    

11. Use existing key pair  
If you already have a key pair configured on your machine you can add the import the public key to your instance  
```
$ aws ec2 import-key-pair --key-name chucks-key --public-key-material fileb://.ssh\id_rsa.pub
```


See [AWS CLI EC2](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-ec2-instances.html) for reference.

__Creating a network interface__  
To create the ENI with private IP addresses
```
$ aws ec2 create-network-interface --subnet-id subnet-aed11acb --groups sg-ad70b8c8 --private-ip-addresses PrivateIpAddress=10.0.0.26,Primary=true PrivateIpAddress=10.0.0.27,Primary=false
```

__Allocating Elastic IP address__  
If you stop the instance in EC2-Classic the EIP is disassociated from the instance, and you have to associate it again when you start the instance. For EC2-VPC the EIP remains associated with the EC2 instance.  
Create an Elastic IP address for VPC
```
$ aws ec2 allocate-address --domain vpc
```
associate the EIP to the Elastic Network Interface (ENI)  
```
$ aws ec2 associate-address --network-interface-id eni-d68df2b3 --allocation-id eipalloc-82e0ffe0
```
The network-interface-id parameter must be obtained from the output of the `create-network-interface` command above.  
The allocation-id parameter must be obtained from the output of that `allocate-address` command above.    


__Attaching the network interface to an instance__  
To attach the ENI to an EC2 instance
```
$ aws ec2 attach-network-interface --network-interface-id eni-5c88f739 --instance-id i-2e7dace3 --device-index 1
```

__Associate the EIP to the ENI__   
To associate the EIP to the ENI
```
$ aws ec2 associate-address --network-interface-id eni-5c88f739 --allocation-id eipalloc-d59f80b7 --private-ip-address 10.0.0.26
```

__Windows Instance__   
[How to connect you instance](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/connecting_to_windows_instance.html?icmpid=docs_ec2_console)
[Free Tier](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsm.page-all-free-tier=1&awsf.Free%20Tier%20Types=tier%2312monthsfree)
[AWS EC2 Forum](https://forums.aws.amazon.com/forum.jspa?forumID=30)  
[EC2 Windows Guide](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/concepts.html)
