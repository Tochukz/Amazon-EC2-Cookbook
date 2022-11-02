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
1. Stop your instance 
    ```
    $ aws ec2 stop-instances --instance-id i-xxxxxxxxxxxxxx
    ```
2. Modify the `instance-type` attribute with the new instance type value 
    ```
    $ aws ec2 modify-instance-attribute --instance-id i-xxxxxxxxxxx --instance-type t2.small
    ```
3. Start the instance back again 
    ```
    $  aws ec2 start-instances --instance-id i-xxxxxxxxxxx
    ```
    
__For incompatible instance type__    
If the instance configuration is not compatible with the instance type you want, then: 
1. Launch a new instance with the configuration that is compatible with the instance type you want. 
2. Migrate your application to the new instance.  
* Backup your data in your original instance 
* Attach any EBS volumes that were attached to your original instance onto the new instance.
* Install your application and any software on your new instance.
* Restore any data.  
* If an Elastic IP address was used in the original instance, associate the Elastic IP address with your new instance.  

__Resources__  
[Change instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html)  
[Get recommendation for an instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-recommendations.html)   
[Compatibility for changing the instance type PDF](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/resize-limitations.html)