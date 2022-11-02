# Change EC2  instance type 
### Change the instance type of a running EC2 instance 
Before you can change the instance type of an existing instance, 
you need to check if the instance configuration is compatible with the instance type that you want. 
Compatibility is determined in the following ways: 
* Virtualization type
* Architecture 
* Network
* Network cards  
* Enhanced networking
* NVMe
* AMI   

__For compatible instance type__  
Note that you can't change the instance type 
* if hibernation is enabled for the instance.  
* if the instance is a _Spot instance_  
For Auto Scaling group, you need suspend the scaling processes for the group while you're changing the instance type.  

To change the instance type to a compatible instance type 
1. Get the instance-id for your instance 
    ```
    $ aws ec2 describe-instances 
    ```
2. Stop your instance 
    ```
    $ aws ec2 stop-instances --instance-id i-xxxxxxxxxxxxxx
    ```
3. Modify the `instance-type` attribute with the new instance type value 
    ```
    $ aws ec2 modify-instance-attribute --instance-id i-xxxxxxxxxxx --instance-type t2.small
    ```
4. Start the instance back again 
    ```
    $  aws ec2 start-instances --instance-id i-xxxxxxxxxxx
    ```
    
__For incompatible instance type__    
If the instance configuration is not compatible with the instance type you want, then: 
1. Create a snapshop of your instances's EBS volume
    ```
    $ aws ec2 create-snapshot --volume-id  vol-xxxxxxxx --description "volume of my Amazon linux"
    ``` 
2. Create an EBS volume using the snapshot created from the previous step
    ```
    $  aws ec2 create-volume --availability-zone eu-west-2b --size 30 --snapshot-id snap-xxxxxxxxxxx
    ```
3. Launch a new EC2 instance using the new EBS volume
    ```
    $ aws ec2 run-instances --image-id ami-0648ea225c13e0729 \
                            --key-name AmazonLinux2 \
                            --security-group-ids sg-097302308e8550121 \     
                            --instance-type t2.small \
                            --block-device-mappings file://block-devices.json
                             
    ```
    See the  

### Changing the instance storage volume of EC2 instance
__Change the instance EBS volume size__   
1. Stop the instance 
    ```
    $ aws ec2 stop-instances --instance-id i-xxxxxxxxxxxxx
    ```
2. Get the EBS volume ID of your instance
    ```
    $  aws ec2 describe-volumes
    ```
3. Create the snapshop of the volumn(optional)
    ```
    $ aws ec2 create-snapshot --volume-id  vol-xxxxxxxx --description "volume of my Amazon linux"
    ```
4. Increase the instance volumn size
    ````
    $ aws ec2 modify-volume --volume-id vol-xxxxxxxxxx --size 16
    ````

__Resources__  
[Change instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html)  
[Get recommendation for an instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-recommendations.html)   
[Compatibility for changing the instance type PDF](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/resize-limitations.html)  
[Create Amazon EBS snapshots](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-snapshot.html)    
[Create an Amazon EBS volume](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html)
