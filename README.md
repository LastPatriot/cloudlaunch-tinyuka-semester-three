# CloudLaunch AWS Project Submission

This repository contains the solution for the CloudLaunch project, which demonstrates fundamental AWS concepts, including S3, IAM, and VPC. The project was completed as part of my cloud computing curriculum at AltSchool Africa.

For context, please see project instructions [here](https://docs.google.com/document/d/11KnEuI-E_19XQq2_xuaJYt4A4EbaMp9_vb9ogCcE5l8/edit?usp=sharing).

## See the live project first

*  [S3 static website url](https://cloudlaunch-site-bucket-opeyemi.s3.us-east-1.amazonaws.com/index.html)
*  [CloudFront URL](https://dfqpyyxrcehbf.cloudfront.net)

## Project Outline

* [Project Objectives](#objectives)
* [Project Overview](#overview)
* [AWS Services used](#services)
* [S3 Buckets](#buckets)
* [Static site and CloudFront Distribution URL](#static)
* [IAM User & Policy](#iam)
* [VPC Design for CloudLaunch Environment](#vpc)
* [Internet Gateway and Route Tables](#igw)
* [Security Groups](#sg)
* [Screenshots of the rendered page in a web browser](#render)

## Project Objectives <a id="objectives"></a>

*   **Deploy and secure a static website on AWS S3.**
*   **Implement least privilege access control using custom IAM policies for S3 and VPC resources.**
*   **Design and configure a secure, segmented VPC environment with appropriate subnets and routing.**
*   **Utilize security groups to enforce granular network traffic policies.**
*   **Demonstrate the integration of S3, IAM, and VPC services for a foundational cloud deployment.**

## Project Overview <a id="overview"></a>

This project was divided into two main tasks:

*   **Task 1:** Hosting a static website on Amazon S3 and creating a specific IAM user with restricted permissions for managing S3 buckets.
*   **Task 2:** Designing a secure and logically separated Virtual Private Cloud (VPC) environment for future application deployment.

# AWS Service(s) used <a id="services"></a>

*   **Amazon S3 (Simple Storage Service)**: To host the static website, storing private documents, and demonstrating various access control levels.
*   **AWS IAM (Identity and Access Management)**: To create users, defining custom policies, and managing permissions for accessing AWS resources.
*   **Amazon VPC (Virtual Private Cloud)**: To create a logically isolated network environment in AWS.
*   **EC2 (Elastic Compute Cloud)**: While no EC2 instances were launched, the IAM policy's `ec2:Describe*` permissions were given to indicate that there would be an interaction with underlying VPC networking components (subnets, route tables, security groups) which are managed under the EC2 service.
*   **AWS CloudFront**: Used for distributing the static website content globally. A key feature implemented is its ability to redirect HTTP traffic to HTTPS, enhancing security.

---


## Task 1: S3 Static Website Hosting & IAM Access Control <a id="buckets"></a>

This task involved creating three S3 buckets with specific configurations and an IAM user with a custom policy to manage access.

### 1.1 S3 Buckets

I created three S3 buckets to fulfill the project requirements:

1.  **`cloudlaunch-site-bucket-opeyemi`**: This bucket hosts a simple static website and is configured for public read-only access.
    *   *Configuration:* Static website hosting was enabled and public read access was granted via a bucket policy.
    *   <img width="1520" height="945" alt="Screenshot 2025-08-25 at 06 44 23" src="https://github.com/user-attachments/assets/065a40e7-fbb8-49bb-90db-e8ed20d311c0" />


2.  **`cloudlaunch-private-bucket-opeyemi`**: This bucket is used for storing internal, private documents. It is not publicly accessible.
    *   *Configuration:* Block Public Access enabled, no public read permissions.
    *   <img width="1520" height="825" alt="Screenshot 2025-08-25 at 06 45 30" src="https://github.com/user-attachments/assets/70479dff-9437-4fc5-b438-3a36fd681755" />


3.  **`cloudlaunch-visible-only-bucket-opeyemi`**: This bucket is also private, and its purpose is to demonstrate an IAM permission where a user can see the bucket exists but cannot access its contents.
    *   *Configuration:* Block Public Access enabled, IAM policy allows `ListBucket` but denies object access.
    *   <img width="1520" height="825" alt="Screenshot 2025-08-25 at 06 46 14" src="https://github.com/user-attachments/assets/26d10e73-792b-4c7f-9884-1dedbdc65de3" />


### 1.2 Static Site URL <a id="static"></a>

The static website is publicly accessible via the following S3 static website hosting URL:

*	`[https://cloudlaunch-site-bucket-opeyemi.s3.us-east-1.amazonaws.com/index.html]`
*	<img width="1520" height="825" alt="Screenshot 2025-08-25 at 06 47 37" src="https://github.com/user-attachments/assets/804fc441-f40d-4287-948a-c1ac06cd003a" />

---

**CloudFront Distribution URL:**

If you configured a CloudFront distribution, please include the URL here:

*	`[https://dfqpyyxrcehbf.cloudfront.net]`

---

### 1.3 IAM User & Policy <a id="iam"></a>

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
*   <img width="1520" height="857" alt="Screenshot 2025-08-25 at 06 55 04" src="https://github.com/user-attachments/assets/847edb44-53ed-4708-896f-f32d4f71046a" />


---

## Task 2: VPC Design for CloudLaunch Environment <a id="vpc"></a>

This task involved creating a VPC named `cloudlaunch-vpc` to establish a secure and isolated network environment for future application deployment.

### 2.1 VPC Details

*   **VPC Name**: `cloudlaunch-vpc`
    *   <img width="1549" height="808" alt="Screenshot 2025-08-25 at 06 57 50" src="https://github.com/user-attachments/assets/7020a121-96ef-457a-8b4c-1443bb257b47" />

*   **CIDR Block**: `10.0.0.0/16`

### 2.2 Subnets 

Three subnets were created within the VPC, each with a specific purpose and CIDR block:

1.  **Public Subnet**:
    *   CIDR Block: `10.0.1.0/24`
    *   Purpose: Intended for public-facing resources like load balancers.
    *   <img width="1549" height="862" alt="Screenshot 2025-08-25 at 06 59 36" src="https://github.com/user-attachments/assets/9d8b3fb0-c149-4076-8038-d829d91eb748" />

2.  **Application Subnet**:
    *   CIDR Block: `10.0.2.0/24`
    *   Purpose: A private subnet for application servers.
    *   <img width="1549" height="848" alt="Screenshot 2025-08-25 at 07 01 34" src="https://github.com/user-attachments/assets/87e9f4de-09f2-4b0b-b337-418bea39aebd" />

3.  **Database Subnet**:
    *   CIDR Block: `10.0.3.0/28`
    *   Purpose: A highly restricted private subnet for database services.
    *   <img width="1549" height="848" alt="Screenshot 2025-08-25 at 07 00 55" src="https://github.com/user-attachments/assets/b08fb5dd-d00d-4548-9feb-a4d82f83c891" />

### 2.3 Internet Gateway and Route Tables <a id="igw"></a>

*   **Internet Gateway**: An Internet Gateway named `cloudlaunch-igw` was created and attached to the `cloudlaunch-vpc`.
    *   <img width="1549" height="848" alt="Screenshot 2025-08-25 at 07 02 45" src="https://github.com/user-attachments/assets/8bf49c09-4425-413a-8a80-e66c830879e0" />


*   **Route Tables**:
    1.  **`cloudlaunch-public-rt`**: Associated with the public subnet. This route table has a default route (`0.0.0.0/0`) pointing to the `cloudlaunch-igw`, allowing internet access.
    2.  **`cloudlaunch-app-rt`**: Created for the application subnet. This route table does **not** have an internet route, ensuring this subnet remains fully private.
    3.  **`cloudlaunch-db-rt`**: Created for the database subnet. This route table also does **not** have an internet route, ensuring this subnet remains fully private.
	*	<img width="1549" height="307" alt="Screenshot 2025-08-25 at 07 04 35" src="https://github.com/user-attachments/assets/8872da4e-ba95-4e3a-989c-f2a4a4fccea7" />


### 2.4 Security Groups <a id="sg"></a>

Two security groups were created to control network traffic within the VPC:

1.  **`cloudlaunch-app-sg`**:
    *   Purpose: Controls traffic for application servers.
    *   Inbound Rule: Allows HTTP (port 80) traffic. The source is restricted to the VPC's CIDR block (`10.0.0.0/16`).
    *   <img width="1549" height="679" alt="Screenshot 2025-08-25 at 07 08 10" src="https://github.com/user-attachments/assets/40a95297-36ce-421d-84ac-41af466fbdda" />


2.  **`cloudlaunch-db-sg`**:
    *   Purpose: Controls traffic for the database.
    *   Inbound Rule: Allows MySQL (port 3306) traffic. The source is the security group ID of `cloudlaunch-app-sg`, ensuring only application servers can communicate with the database.
    *   <img width="1549" height="679" alt="Screenshot 2025-08-25 at 07 07 19" src="https://github.com/user-attachments/assets/d88fd95f-cf44-423f-b799-357db28274e3" />

## Screenshots of the rendered page in a web browser <a id="render"></a>
*	<img width="1798" height="1071" alt="Screenshot 2025-08-25 at 07 17 20" src="https://github.com/user-attachments/assets/31d45106-0de8-4788-8a2b-49d1dd291d58" />

---

## Contact

For inquiries, you can reach Opeyemi Oluwadare at:
* **Phone:** 08167877918
* **Email:** oluwadareopeyemis1@gmail.com
