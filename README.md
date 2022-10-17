# Amazon EC2 Cookbook  (2015)
__By Sekhar Reddy and Autobindo Sarkar__  

## Chapter 1: Selecting and Configuring Amazon EC2 Instances  
__Introduction__  
You need to ask yourself several questions in order to choose the right AWS EC2 instance for meeting your requirements.
1. What is the primary purpose of the EC2 instance being provisioned?
2. What is the duration of your need for a particular machine?
3. Do you need high performance storage?
4. Should you go for dedicated or shared tenancy?
5. Will the machine be used for compute-intensive or memory-intensive processing?
6. What are the scalability, availability, and security requirements?
7. What are your networking requirements?

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

-- Todo: Continue fron page 27 - Allocating Elastic IP addresses

__Windows Instance__   
[How to connect you instance](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/connecting_to_windows_instance.html?icmpid=docs_ec2_console)
[Free Tier](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsm.page-all-free-tier=1&awsf.Free%20Tier%20Types=tier%2312monthsfree)
[AWS EC2 Forum](https://forums.aws.amazon.com/forum.jspa?forumID=30)  
[EC2 Windows Guide](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/concepts.html)


## Chapter 2: Configuring and Securing a Virtual Private Cloud  

Todo: Continue later

## Chapter 3: Managing AWS Resources Using AWS CloudFormation   
[AWS CloudFormation Docs](https://docs.aws.amazon.com/cloudformation/index.html)  
[AWS CloudFornation User Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)  

__Introduction__   
A template is essentially a JavaScript Object Notation (JSON) file. Using AWS CloudFormation, you can implement version control for your infrastructure and define new templates, or use existing ones, to create a stack (a running instance of your template). If the stack creation process fails, then the entire process gets rolled back.

__Cloudformation Designer__   
To start designing your own templates with AWS CloudFormation Designer, go to [CloudFormation Designer](https://eu-west-2.console.aws.amazon.com/cloudformation/designer/home?region=eu-west-2#)   

__Change sets__  
Before making changes to your resources, you can generate a change set, which is a summary of your proposed changes. Change sets allow you to see how your changes might impact your running resources, especially for critical resources, before implementing them.

__Cloudformation and S3__  
If you specify a template file stored locally, CloudFormation uploads it to an S3 bucket in your AWS account.
You can use your own bucket and manage its permissions by manually uploading templates to Amazon S3. Then whenever you create or update a stack, specify the Amazon S3 URL of a template file.

__Updating a stack with change sets__   
To update a stack, create a change set by submitting a modified version of the original stack template, different input parameter values, or both. CloudFormation compares the modified template with the original template and generates a change set. The change set lists the proposed changes. After reviewing the changes, you can start the change set to update your stack or you can create a new change set.

Learn more about [Private link and VPC Networks](https://docs.aws.amazon.com/vpc/latest/userguide/#what-is-privatelink)  

__Example 1__  
The CloudFormation template is for an autoscale web application. It defines :
- Elastic Load Balancer
- Launching configurations  
- Auto scaling groups   
- Cloud Watch Alarms  
- Security Groups  
It also includes commands to
- retrieve AWS resource description
- get access to stack events
- send event notification to an SNS topic
See the template file `WebAutoScale.json`   

A template is divided into multiple sections— _resources_, _parameters_, _mappings_, and _outputs_.
1. resources section contains the list of all the resources and their properties
2. parameters section specify the parameters that can be customized at the stack creation time
3. mappings section allows you to perform conditional setting of properties that are evaluated in a similar manner as a look up table statement
4. the outputs section is used to return important data to the user based on the resources created

Validate the template   
```
$ aws cloudformation validate-template --template-body file://AutoScaleStack.json
```
Create a stack using the template  
```
$ aws cloudformation create-stack --stack-name WebAutoScale --template-body file://AutoScaleStack.json --parameters ParameterKey=NameForNewInstances,ParameterValue=MyApacheServerInstance
```  
Get the resources description
```
$ aws cloudformation describe-stack-resources --stack-name WebAutoScale
```
Get the list of events of the stack
```
$ aws cloudformation describe-stack-events --stack-name WebAutoScale
```

__Deleting a stack__  
To delete an existing stack
```
$ aws cloudformation delete-stack --stack-name WebAutoScale
```  
If you want to delete a stack but want to retain some resources in that stack, you can use a deletion policy to retain those resources. Learn more about [deletion policy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-deletionpolicy.html).

__Templates__  
[WordPress sample template](https://s3.us-west-2.amazonaws.com/cloudformation-templates-us-west-2/WordPress_Single_Instance.template)   


__Resources__  
[AWS SDKs and Code Examples](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sdk-general-information-section.html)   
[AWS CloudFormation Templates](https://aws.amazon.com/cloudformation/resources/templates/)   

## Chapter 6: Using AWS Data Service
Todo: Read this later

### Using Amazon RDS  
[Amazon RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)  

__Create a RDS DB instance using default VPC__   
1. Create the DB instance
```
$ aws rds create-db-instance --db-instance-identifier MyPostgresInstance --db-name MyDbName --db-instance-class db.t3.micro --engine postgres  --master-username MyUsername --master-user-password MyPassword --allocated-storage 20
```  
Copy the `VpcSecurityGroupId` from the returned JSON response.
2. Add a firewall rule to the security group of the newly created instance
```
aws ec2 authorize-security-group-ingress --group-id sg-xxxxxxxxxx --protocol tcp --port 5432 --cidr 0.0.0.0/0
```  
This CIDR makes the instanced publicly available.  
If you did failed to copy the VpcSecurityGroupId from the response the `create-db-instance` operation you can do
```
$ aws rds describe-db-instances
```
and copy the `VpcSecurityGroupId` from the relevant DB instance.  
3. Copy your DB instance host endpoint
```
$ aws rds describe-db-instances >> dbinstances.json
```
look for the `Endpoint` object and copy the value of `Address` property.
4. To delete the DB instance
```
$ aws rds delete-db-instance --db-instance-identifier MyPostgresInstance --skip-final-snapshot
```
If you want to save a snapshot of the DB, then use the `--final-db-snapshot-identifier` flag instead
```
$ aws rds delete-db-instance --db-instance-identifier MyPostgresInstance  --final-db-snapshot-identifier myDbInstanceFinalSnapshot
```

__Create a RDS DB instance using your custom VPC__    
1. Copy your VPC ID
```
$ aws ec2 describe-vpcs
```
Your custom VPC should have `IsDefault: false` in the JSON response.
If you do not already have a custom VPC, create one
```
$ aws ec2 create-vpc --cidr-block 10.0.0.0/16
```
and copy the VPC ID.
2. Create a DB subnet group



2. Create security group in the VPC
```
$ aws ec2 create-security-group --group-name PostgresSG --description "Security group for PostgreSQL instance" --vpc-id vpc-my-vpc-id
```
3. Add a rule to the security group
```
$ aws ec2 authorize-security-group-ingress --group-id sg-0aaa0cf72d9b56958 --protocol tcp --port 5432 --cidr 0.0.0.0/0
```  
This rule permits public access over port `5432` which is the port used by PosgresSQL DB instance
4. Create the DB instance
```
$ aws rds create-db-instance --db-instance-identifier MyPostgresInstance --db-name add3_dev_db --db-instance-class db.t3.micro --engine postgres --vpc-security-group-ids sg-0aaa0cf72d9b56958 --master-username dbWebChain --master-user-password passAddArch2118 --allocated-storage 20
```

__Learn More__    
[Amazon VPC VPCs and Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.html)  
[AWS CLI RDS Guide](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/rds/index.html)
