1. Create the S3 bucket. NOTE: This can be any name, but if you plan on redirecting non-www to www, or vice versa, then make two buckets with the domain names (domain.com, AND www.domain.com).
2. Make the S3 bucket(s) public. See [Granting read-only permission to an anonymous user](https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-bucket-policies.html)
3. Activate S3 static website hosting on the bucket(s).
4. Add the website to the primary bucket you plan on using.
5. Go to ACM in the us-east-1 region and create a certificate for the domain name.