# AWS Compute Optimizer 
[User Guide](https://docs.aws.amazon.com/compute-optimizer/latest/ug/what-is-compute-optimizer.html)
### Introduction  
AWS Compute Optimizer is a service that analyzes the configuration and utilization metrics of your AWS resources. 
It reports whether your resources are optimal, and generates optimization recommendations to reduce the cost and improve the performance of your workloads.  

### Supported resources
* Amazon Elastic Compute Cloud (Amazon EC2) instances
* Amazon EC2 Auto Scaling groups
* Amazon Elastic Block Store (Amazon EBS) volumes
* AWS Lambda functions  

### Requirements
> __Service:__ CloudWatch Metric  
__Requirement:__ 30 consecutive hours of CloudWatch metric data from your resource. Not applicable to Lambda functions  

> __Service:__ Amazon EC2       
__Reqirement:__ The instance type must be one supported by AWS compute optimizer. Compute Optimizer currently generates recommendations for C, D, H, I, M, R, T, X, and z instance types.

> __Service:__ Auto Scaling group   
__Requirement:__ The Auto scaling group must: 
> * run supported instance types 
> * run a single type of instance (not mixed instance types) 
> * have no scaling policy attached.
> * have no overrides configured.  

> __Service:__ Amazon EBS volume
__Requirement:__ General Purpose SSD (gp2 and gp3) and Provisioned IOPS SSD (io1 and io2) EBS volume types that are attached to an instance for atleast 30 cosecutive hours.    

> __Service:__ Lambda function  
__Requirement:__  
> * Configured memory is less than or equal to 1,792 MB
> * Functions are invoked at least 50 times in the last 14 days

### Getting started 
Before you can use the service, you must opt in or out.  
By opting in, you're authorizing Compute Optimizer to analyze the specifications and utilization metrics of your AWS resources.  
Your account must be the an account type that is supported by AWS compute optimizer.  

To opt in your individual account: 
```
$ aws compute-optimizer update-enrollment-status --status Active
```  
To opt in the management account of an organization and include all member accounts within the organization: 
```
$ aws compute-optimizer update-enrollment-status --status Active --include-member-accounts
```