# AWS S3

## What?

- Amazon Simple Storage Service (S3) is a managed service for storing files and other data as **objects**.
- Think of S3 as a virtually unlimited warehouse: **buckets** are storage containers, and **objects** are the files stored inside them.
- An object contains the file data, metadata, and a unique key that acts like its path or name.
- S3 is object storage, not a normal disk or database. Applications access objects through APIs or URLs rather than attaching S3 like a local drive.

## How?

- Create a bucket with a globally unique name and choose an AWS Region.
- Upload objects such as images, videos, documents, backups, and build artifacts.
- Use object keys such as `images/logo.png` to organize files into logical folders.
- Control access with IAM policies, bucket policies, and object ownership settings.
- Enable versioning to keep older versions when objects are replaced or deleted.
- Configure lifecycle rules to move older objects to cheaper storage classes or delete them automatically.
- Enable encryption, logging, and access monitoring according to the application's security requirements.

## Why?

- S3 provides highly durable storage without managing disks or storage servers.
- It scales automatically as the number and size of objects grow.
- It is useful for static assets, backups, data lakes, logs, media, and deployment artifacts.
- Storage classes help balance access speed and cost for frequently or rarely accessed data.
- S3 can integrate with CloudFront, CloudWatch, Lambda, and many other AWS services.

> **Analogy:** A bucket is like a warehouse, an object is a package, and the object key is the label that tells you where to find the package.

> **Typical flow:** Application -> S3 bucket -> object upload/download -> access controlled by IAM and bucket policies

## Steps to create an S3 bucket

1. Open the AWS Console, search for **S3**, and select **Create bucket**.
2. Under **General configuration**, enter a globally unique **Bucket name**, such as `my-app-assets-123456`.
	- Bucket names must be unique across all AWS accounts.
	- Choose a name that does not contain sensitive information because bucket names can appear in URLs and logs.
3. Choose an AWS **Region**.
	- Choose a Region close to the users or services that access the data.
	- Keeping S3 in the same Region as EC2, ECS, or Lambda can reduce latency and data-transfer costs.
4. Under **Object Ownership**, choose **ACLs disabled - Bucket owner enforced** unless an older application specifically requires ACLs.
	- This makes the bucket owner responsible for all objects and simplifies access control through IAM and bucket policies.
5. Under **Block Public Access settings**, keep all options enabled unless public access is explicitly required.
	- These settings prevent accidental public exposure through bucket or object policies.
	- For public static content, use a deliberate design such as CloudFront and carefully reviewed policies rather than disabling protection casually.
6. Under **Bucket Versioning**, choose one:
	- **Enable:** Keeps previous object versions and helps recover from accidental deletion or overwrites.
	- **Disable:** Uses less storage, but deleted or replaced objects cannot be recovered through version history.
7. Under **Tags**, add optional key-value tags such as `Environment=dev` and `Application=my-app`.
	- Tags help with cost allocation, searching, automation, and resource management.
8. Under **Default encryption**, keep server-side encryption enabled:
	- **SSE-S3:** AWS manages the encryption keys and is the simplest default.
	- **SSE-KMS:** Uses AWS KMS for more control, auditing, and key-management policies; it may add KMS request and key costs.
9. Review **Advanced settings** if needed:
	- **Object Lock:** Helps prevent objects from being deleted or changed during a retention period; enable it only when compliance or immutable backups are required.
	- **Event notifications:** Configure later if uploads should trigger Lambda, SQS, or SNS actions.
10. Review the settings and select **Create bucket**.
11. Open the bucket, select **Upload**, choose files or folders, and configure the storage class and optional metadata.
12. Configure access separately using IAM policies, bucket policies, or an application role. Avoid making the bucket public unless there is a clear requirement.

> **Beginner-friendly defaults:** Choose a nearby Region, disable ACLs, keep Block Public Access enabled, enable versioning for important data, use SSE-S3 encryption, and add tags for the environment and application.

