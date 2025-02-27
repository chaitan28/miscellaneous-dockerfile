# Acount B Needs s3 bucket which is Account A.
### Account A:
1. Create an IAM Policy in Account A (S3 Bucket Owner)<br>
2. Go to AWS IAM Console in Account A.<br>
Create a new policy (AccountA-S3-Access) with the following JSON:<br>
```sh
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::whytebatl.com",
                "arn:aws:s3:::whytebatl.com/*"
            ]
        }
    ]
}
```
3. Attach this policy to the S3 bucket in Account A.<br>

### Account B:
1. Go to AWS IAM Console in Account B <br>
2. create a aws managed policy or inline policy with name cross-s3-account:
```sh
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::whytebatl.com",
                "arn:aws:s3:::whytebatl.com/*"
            ]
        }
    ]
}
```
3. Create a new IAM Role (EC2-S3-Access): <br>
- Trusted entity: AWS Service
- Use case: EC2
- Attach the above policy(cross-s3-account)

4. Attach the IAM role to your EC2 instance in Account B <br>