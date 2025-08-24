CloudLaunch AWS Project Submission

This repository contains the solution for the CloudLaunch project, which demonstrates my understanding of fundamental AWS concepts, including S3, IAM, and VPC. The project was completed as part of my cloud computing curriculum.

## Project Overview

This project was divided into two main tasks:

*   **Task 1:** Hosting a static website on Amazon S3 and creating a specific IAM user with restricted permissions for managing S3 buckets.
*   **Task 2:** Designing a secure and logically separated Virtual Private Cloud (VPC) environment for future application deployment.

---

## Task 1: S3 Static Website Hosting & IAM Access Control

This task involved creating three S3 buckets with specific configurations and an IAM user with a custom policy to manage access.

### 1.1 S3 Buckets

I created three S3 buckets to fulfill the project requirements:

1.  **`cloudlaunch-site-bucket-opeyemi`**: This bucket hosts a simple static website and is configured for public read-only access.
    *   *Configuration:* Static website hosting enabled, public read access granted via a bucket policy.
    *   ![Screenshot Placeholder](Configuration of cloudlaunch-site-bucket for static website hosting and public access)

2.  **`cloudlaunch-private-bucket-opeyemi`**: This bucket is used for storing internal, private documents. It is not publicly accessible.
    *   *Configuration:* Block Public Access enabled, no public read permissions.
    *   ![Screenshot Placeholder](Configuration of cloudlaunch-private-bucket with public access blocked)

3.  **`cloudlaunch-visible-only-bucket-opeyemi`**: This bucket is also private, and its purpose is to demonstrate an IAM permission where a user can see the bucket exists but cannot access its contents.
    *   *Configuration:* Block Public Access enabled, IAM policy allows `ListBucket` but denies object access.
    *   ![Screenshot Placeholder](Configuration of cloudlaunch-visible-only-bucket with restricted IAM access)

### 1.2 Static Site URL

The static website is publicly accessible via the following S3 static website hosting URL:

`[S3 Static Site Link](https://cloudlaunch-site-bucket-opeyemi.s3.us-east-1.amazonaws.com/index.html)`

---

**CloudFront Distribution URL:**

If you configured a CloudFront distribution, please include the URL here:

`[CloudFront URL](https://dfqpyyxrcehbf.cloudfront.net)`

---

### 1.3 IAM User & Policy

A user named `cloudlaunch-user` was created with a custom IAM policy attached. This policy grants the user specific, limited permissions across the three S3 buckets and read-only access to the VPC components.

**User Permissions Summary:**

*   **`cloudlaunch-site-bucket`**: `ListBucket` and `GetObject` permissions.
*   **`cloudlaunch-private-bucket`**: `ListBucket`, `GetObject`, and `PutObject` permissions. `DeleteObject` is **not** allowed.
*   **`cloudlaunch-visible-only-bucket`**: `ListBucket` permission only. No access to objects.
*   **VPC Components**: Read-only (`ec2:Describe*`, `ec2:Get*`) access to the VPC, subnets, route tables, and security groups.

---

### 1.4 Custom IAM Policy JSON

The following JSON document was used to define the custom policy attached to `cloudlaunch-user`:

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "ListAllCloudlaunchBuckets",
			"Effect": "Allow",
			"Action": "s3:ListBucket",
			"Resource": [
				"arn:aws:s3:::cloudlaunch-site-bucket-opeyemi",
				"arn:aws:s3:::cloudlaunch-private-bucket-opeyemi",
				"arn:aws:s3:::cloudlaunch-visible-only-bucket-opeyemi"
			]
		},
		{
			"Sid": "AllowGetObjectOnSiteBucket",
			"Effect": "Allow",
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::cloudlaunch-site-bucket-opeyemi/*"
		},
		{
			"Sid": "AllowGetPutOnPrivateBucket",
			"Effect": "Allow",
			"Action": [
				"s3:GetObject",
				"s3:PutObject"
			],
			"Resource": "arn:aws:s3:::cloudlaunch-private-bucket-opeyemi/*"
		},
		{
			"Sid": "DenyObjectAccessToVisibleOnlyBucket",
			"Effect": "Deny",
			"Action": [
				"s3:GetObject",
				"s3:PutObject",
				"s3:DeleteObject"
			],
			"Resource": "arn:aws:s3:::cloudlaunch-visible-only-bucket-opeyemi/*"
		},
		{
			"Sid": "ReadOnlyVpcAccess",
			"Effect": "Allow",
			"Action": [
				"ec2:Describe*",
				"ec2:Get*",
				"ec2:List*"
			],
			"Resource": "*"
		}
	]
}
```
*   ![Screenshot Placeholder](IAM policy JSON for cloudlaunch-user)

---

## Task 2: VPC Design for CloudLaunch Environment

This task involved creating a VPC named `cloudlaunch-vpc` to establish a secure and isolated network environment for future application deployment.

### 2.1 VPC Details

*   **VPC Name**: `cloudlaunch-vpc`
    *   ![Screenshot Placeholder](Creation of cloudlaunch-vpc in AWS console)
*   **CIDR Block**: `10.0.0.0/16`

### 2.2 Subnets

Three subnets were created within the VPC, each with a specific purpose and CIDR block:

1.  **Public Subnet**:
    *   CIDR Block: `10.0.1.0/24`
    *   Purpose: Intended for public-facing resources like load balancers.
    *   ![Screenshot Placeholder](Configuration of the public subnet in cloudlaunch-vpc)
2.  **Application Subnet**:
    *   CIDR Block: `10.0.2.0/24`
    *   Purpose: A private subnet for application servers.
    *   ![Screenshot Placeholder](Configuration of the application subnet in cloudlaunch-vpc)
3.  **Database Subnet**:
    *   CIDR Block: `10.0.3.0/28`
    *   Purpose: A highly restricted private subnet for database services.
    *   ![Screenshot Placeholder](Configuration of the database subnet in cloudlaunch-vpc)

### 2.3 Internet Gateway and Route Tables

*   **Internet Gateway**: An Internet Gateway named `cloudlaunch-igw` was created and attached to the `cloudlaunch-vpc`.
    *   ![Screenshot Placeholder](Creation and attachment of cloudlaunch-igw to cloudlaunch-vpc)

*   **Route Tables**:
    1.  **`cloudlaunch-public-rt`**: Associated with the public subnet. This route table has a default route (`0.0.0.0/0`) pointing to the `cloudlaunch-igw`, allowing internet access.
        *   ![Screenshot Placeholder](Configuration of cloudlaunch-public-rt with default route to IGW)
    2.  **`cloudlaunch-app-rt`**: Created for the application subnet. This route table does **not** have an internet route, ensuring this subnet remains fully private.
        *   ![Screenshot Placeholder](Configuration of cloudlaunch-app-rt, showing no internet route)
    3.  **`cloudlaunch-db-rt`**: Created for the database subnet. This route table also does **not** have an internet route, ensuring this subnet remains fully private.
        *   ![Screenshot Placeholder](Configuration of cloudlaunch-db-rt, showing no internet route)

### 2.4 Security Groups

Two security groups were created to control network traffic within the VPC:

1.  **`cloudlaunch-app-sg`**:
    *   Purpose: Controls traffic for application servers.
    *   Inbound Rule: Allows HTTP (port 80) traffic. The source is restricted to the VPC's CIDR block (`10.0.0.0/16`).
    *   ![Screenshot Placeholder](Configuration of cloudlaunch-app-sg inbound rules)

2.  **`cloudlaunch-db-sg`**:
    *   Purpose: Controls traffic for the database.
    *   Inbound Rule: Allows MySQL (port 3306) traffic. The source is the security group ID of `cloudlaunch-app-sg`, ensuring only application servers can communicate with the database.
    *   ![Screenshot Placeholder](Configuration of cloudlaunch-db-sg inbound rules, referencing cloudlaunch-app-sg)

---
