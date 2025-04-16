
## Assumption
Something I wasn't quite sure about in AWS is if DeleteObject permission is required when overwriting an object in S3. So I ran a little test to check.

When uploading a file to S3, and overwriting an existing file, we do not need DeleteObject permissions. GetObject and PutObject is all that is required. It could be assumed that we would require DeleteObject permissions as we are overwriting an existing object. However as can be seen below this is not the case.

###  Inline policy used:

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject",
                    "s3:PutObject"
                ],
                "Resource": "arn:aws:s3:::test-env-name-ssec-test-bucket-a/*"
            }
        ]
    }

### Results
(**This is assuming versioning is not enabled for S3 object in the S3 bucket.**)
### Encrypting file locally after downloading it from S3 (GetObject):
➜  ~ openssl enc -aes-256-cbc -salt -in plaintext.txt -out plaintext.txt

enter AES-256-CBC encryption password:

Verifying - enter AES-256-CBC encryption password:

*** WARNING : deprecated key derivation used.

Using -iter or -pbkdf2 would be better.

### Overwrite the existing file in S3:
➜  ~ aws --profile testEnv s3 cp ./plaintext.txt s3://test-env-name-ssec-test-bucket-a/plaintext.txt

upload: ./plaintext.txt to s3://test-env-name-ssec-test-bucket-a/plaintext.txt

### Downloading the original file and making sure it was overwritten / encrypted:

➜  ~ aws --profile testEnv s3 cp s3://test-env-name-ssec-test-bucket-a/plaintext.txt ./plaintext.txt

download: s3://test-env-name-ssec-test-bucket-a/plaintext.txt to ./plaintext.txt

➜  ~ cat plaintext.txt

    Salted__
    
    @_   F  A]Ʃne    ? 4%
As can be seen above, we were able to overwrite an existing S3 object without the need for DeleteObject permissions.

## (Update) Adding an explicit deny for DeleteObject to the policy

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject",
                    "s3:PutObject"
                ],
                "Resource": "arn:aws:s3:::test-env-name-ssec-test-bucket-a/*"
            },
            {
                "Effect": "Deny",
                "Action": [
                    "s3:DeleteObject"
                ],
                "Resource": "arn:aws:s3:::test-env-name-ssec-test-bucket-a/*"
            }
        ]
    }

### Results
robdj in ~/s3tests λ aws --profile testEnv s3 cp ./plaintext.txt s3://test-env-name-ssec-test-bucket-a/plaintext.txt

upload: ./plaintext.txt to s3://test-env-name-ssec-test-bucket-a/plaintext.txt

robdj in ~/s3tests λ aws s3api --profile testEnv delete-object --bucket test-env-name-ssec-test-bucket-a --key plaintext.txt

An error occurred (AccessDenied) when calling the DeleteObject operation: User: arn:aws:iam::xxxxxxxxxxxx:user/bucket_test_user is not authorized to perform: s3:DeleteObject on resource: "arn:aws:s3:::test-env-name-ssec-test-bucket-a/plaintext.txt" with an explicit deny in an identity-based policy

