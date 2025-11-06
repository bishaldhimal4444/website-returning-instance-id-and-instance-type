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

IMDSv2 vs IMDSv1

Originally, there was IMDSv1, where you could directly do:
```
curl http://169.254.169.254/latest/meta-data/instance-id
```
and it would return something like:
```
i-0abc123xyz
```
But IMDSv1 had a security risk — if your app had a bug (like SSRF), someone could trick it into exposing metadata.

So AWS introduced IMDSv2, which adds a security token step.

How IMDSv2 works (step-by-step)

Let’s walk through it simply:

Step 1️⃣ – Ask for a token

You send a PUT request to get a temporary token:
```
curl -X PUT "http://169.254.169.254/latest/api/token" \
     -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"
```
+ -X PUT → means you’re asking for a token.
+ 21600 → how long (in seconds) the token will stay valid (here, 6 hours).

You’ll get back a string, something like:
```
AQAE1234567890EXAMPLE==
```
That’s your token — kind of like a short-lived password.

Step 2️⃣ – Use that token to get metadata

Now you include that token in your next metadata request:
```curl -H "X-aws-ec2-metadata-token: AQAE1234567890EXAMPLE==" \
     http://169.254.169.254/latest/meta-data/instance-id

```
That returns:
```
i-0a12b34c56d78ef90
```
You can reuse that token to fetch other metadata (like instance type, AZ, etc.) until it expires.

Example summary in one view
```
# Get token
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# Use token to fetch instance ID
INSTANCE_ID=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id)

echo "Instance ID is: $INSTANCE_ID"
```
