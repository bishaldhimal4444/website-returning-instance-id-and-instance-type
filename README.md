# website-returning-instance-id-and-instance-type
website returning instance id and instance type

# How It is Happening? 
What is IMDS?

IMDS stands for Instance Metadata Service.
It’s a special internal web server that runs inside every EC2 instance at this IP:
```
http://169.254.169.254
```
You can use it to get information about the instance itself, such as:

- Instance ID
- Instance type
- Availability zone
- Security groups
- IAM role credentials (if attached)

This data is only accessible from inside the EC2 instance.
