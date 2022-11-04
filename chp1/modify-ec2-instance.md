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
    $ aws ec2 stop-instances --instance-id i-xxxxxxx
    ```
3. Modify the `instance-type` attribute with the new instance type of your choice 
    ```
    $ aws ec2 modify-instance-attribute --instance-id i-xxxxxxx --instance-type t2.small
    ```
    Note that you can also use the `modify-instance-attribute` subcommand to modify other attributes of your EC2 instance.
4. Start the instance back again 
    ```
    $  aws ec2 start-instances --instance-id i-xxxxxx
    ```
    
__For incompatible instance type__    
If the instance configuration is not compatible with the instance type you want, then: 
1. Create a snapshop of your existing instances' EBS volume
    ```
    $ aws ec2 create-snapshot --volume-id  vol-xxxxxxxx --description "volume of my Amazon linux"
    ``` 
2. Create a new EBS volume using the snapshot created from the previous step
    ```
    $  aws ec2 create-volume --availability-zone eu-west-2b --size 30 --snapshot-id snap-xxxxxxxxxxx
    ```
3. Launch a new EC2 instance with the configuration of your choice
    ```
    $ aws ec2 run-instances --image-id ami-0648ea225c13e0729 --key-name AmzLinuxKey2 --security-group-ids sg-097302308e8550121 --instance-type t2.small                              
    ```
4. Stop the new EC2 instance 
    ```
    $ aws ec2 stop-instances --instance-id i-0b30a41cb2e12dd59
    ```
5. View all you existing EBS volumes
    ```
    $ aws ec2 describe-volumes 
    ```
    Take note of the volume attached to the newly created EC2 instance and copy the it's volumeId and device value (e.g `/dev/xvda`).  

6. Detach the default EBS volume from the new EC2 instance 
    ```
    $ aws ec2 detach-volume --volume-id vol-xxxxxxxx
    ```
7. Attach the EBS volume created in step 2 to the new EC2 instnace
    ```
    $ aws ec2 attach-volume --volume-id vol-06705fb80fd3a99f5 --instance-id i-xxxxxxx --device /dev/xvda
    ```
8. Start the instance new EC2 instance again
    ```
    $ aws ec2 start-instances --instance-id  i-xxxxxxxxx
    ```
9. Delete the detached EBS volume from step 6.
    ```
    $ aws ec2 delete-volume --volume-id vol-0db2448f3b25f0ae6
    ```
10. SSH into the new instances and make sure all your files are present. 
11. Terminate the old instance 
    ```
    $ aws ec2 terminate-instances --instance-id i-xxxxxxxxxxx
    ```

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
3. Create a snapshop of the volume(optional)
    ```
    $ aws ec2 create-snapshot --volume-id  vol-xxxxxxxx --description "volume of my Amazon linux"
    ```
4. Increase the instance volumn size
    ````
    $ aws ec2 modify-volume --volume-id vol-xxxxxxx --size 16
    ````

__Resources__  
[Change instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html)  
[Get recommendation for an instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-recommendations.html)   
[Compatibility for changing the instance type PDF](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/resize-limitations.html)  
[Create Amazon EBS snapshots](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-snapshot.html)    
[Create an Amazon EBS volume](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html)
