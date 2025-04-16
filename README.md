# Hello World
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


➜  ~ openssl enc -aes-256-cbc -salt -in plaintext.txt -out plaintext.txt
enter AES-256-CBC encryption password:
Verifying - enter AES-256-CBC encryption password:
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.

➜  ~ aws --profile testEnv s3 cp ./plaintext.txt s3://test-env-name-ssec-test-bucket-a/plaintext.txt
upload: ./plaintext.txt to s3://test-env-name-ssec-test-bucket-a/plaintext.txt

➜  ~ aws --profile attacksim s3 cp s3://test-env-name-ssec-test-bucket-a/plaintext.txt ./plaintext.txt
download: s3://test-env-name-ssec-test-bucket-a/plaintext.txt to ./plaintext.txt

➜  ~ cat plaintext.txt
Salted__
@_   F  A]Ʃne    ? 4%
