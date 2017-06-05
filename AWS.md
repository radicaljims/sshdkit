## AWS

1. Create AWS account
2. Create IAM user for aws cli to fiddle with
3. Create S3 bucket for vms
4. Give IAM user access to S3 bucket
  1. I added the IAM user to the broad EC2 and S3 groups (may not be necessary)
  2. I added the security polices described here to the bucket (substituting the IAM user and my S3 bucket): http://docs.aws.amazon.com/AmazonS3/latest/dev/example-walkthroughs-managing-access-example1.html
  3. Later came back to give the user the power to CreateRole by being lazy and adding the IAM user to AdministratorAccess
5. Upload image to S3 bucket
  1. ```aws s3 --region us-east-2 cp aws.img s3://aws-vms```
6. Create vmimport roles and policies: http://docs.aws.amazon.com/vm-import/latest/userguide/vmimport-image-import.html#vmimport-service-role
7. Import image

### CLI

#### Upload img to S3
```bash
aws s3 --region us-east-2 cp aws.img s3://aws-vms
```

#### Create trust policy

```bash
aws iam create-role --role-name vmimport --assume-role-policy-document file://trust-policy.json
```

```json
{
   "Version": "2012-10-17",
   "Statement": [
      {
         "Effect": "Allow",
         "Principal": { "Service": "vmie.amazonaws.com" },
         "Action": "sts:AssumeRole",
         "Condition": {
            "StringEquals":{
               "sts:Externalid": "vmimport"
            }
         }
      }
   ]
}
```

#### Create role policy
```bash
aws iam put-role-policy --role-name vmimport --policy-name vmimport --policy-document file://role-policy.json
```

```json
{
   "Version": "2012-10-17",
   "Statement": [
      {
         "Effect": "Allow",
         "Action": [
            "s3:ListBucket",
            "s3:GetBucketLocation"
         ],
         "Resource": [
            "arn:aws:s3:::aws-vms"
         ]
      },
      {
         "Effect": "Allow",
         "Action": [
            "s3:GetObject"
         ],
         "Resource": [
            "arn:aws:s3:::aws-vms/*"
         ]
      },
      {
         "Effect": "Allow",
         "Action":[
            "ec2:ModifySnapshotAttribute",
            "ec2:CopySnapshot",
            "ec2:RegisterImage",
            "ec2:Describe*"
         ],
         "Resource": "*"
      }
   ]
}
```

#### Convert S3 image to AMI

```bash
aws ec2 import-snapshot --description "Linuxkit Appliance" --disk-container file://aws_container.json
```

```json
{
    "Description": "Linuxkit Appliance",
    "Format": "raw",
    "UserBucket": {
        "S3Bucket": "aws-vms",
        "S3Key": "aws.img"
    }
}
```

#### Register the image (SnapshotId is returned from the import-snapshot command above)

```bash
aws ec2 --region us-east-2 register-image --name "Linuxkit Appliance #1" --architecture x86_64 --virtualization-type hvm --root-device-name /dev/sda1 --block-device-mappings '[{"DeviceName": "/dev/sda1", "Ebs": {"SnapshotId": "snap-058563f9030bb0d54", "VolumeType": "gp2"}}]'
```

#### Create a security group to open up ssh/http

```bash
aws ec2 --region us-east-2 create-security-group --group-name linuxkit-sg --description "SG for Linuxkit"
aws ec2 --region us-east-2 authorize-security-group-ingress --group-name linuxkit-sg --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 --region us-east-2 authorize-security-group-ingress --group-name linuxkit-sg --protocol tcp --port 80 --cidr 0.0.0.0/0
```

#### Launch the instance (the PEM file is got by creating a key pair in the AWS console)

```bash
aws ec2 --region us-east-2 run-instances --image-id ami-b51533d0 --count 1 --instance-type t2.micro --security-group-ids sg-a2f4decb --key-name amazon-dr
```

#### SSH into instance (the IP address is got from the EC2 instance list)

```bash  
ssh -i ~/Downloads/amazon-dr.pem root@$(IP)
```

#### Once connected, use `nsenter` to move into the container context

```bash
nsenter -t 1 -m -u -n -i ash
```
