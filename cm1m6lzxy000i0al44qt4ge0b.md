---
title: "Accessing AWS DocumentDB with Dynamic Secrets via HashiCorp Vault"
datePublished: Sat Sep 28 2024 13:22:32 GMT+0000 (Coordinated Universal Time)
cuid: cm1m6lzxy000i0al44qt4ge0b
slug: accessing-aws-documentdb-with-dynamic-secrets-via-hashicorp-vault
tags: aws, devops, terraform

---

[*DocumentDB*](https://aws.amazon.com/documentdb/) *is a fully managed NoSQL database service that has* [*compatibility with MongoDB*](https://aws.amazon.com/documentdb/faqs/#:~:text=What%20does%20%22MongoDB%2Dcompatible%22,with%20little%20or%20no%20changes.)*.*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727529311058/acd8ed47-680f-4eee-989c-66df47db584a.png align="center")

## Introduction

In today's digital age, keeping our data secure is more important than ever. One of the most effective ways to manage sensitive information, like database credentials, is by using dynamic secrets. In this guide, we'll walk through how to set up AWS DocumentDB with dynamic secrets using HashiCorp Vault. By the end, you’ll have a secure database setup and a clear understanding of how dynamic secrets work.

### What Are Dynamic Secrets?

Dynamic secrets are temporary credentials generated on-demand, providing access only when needed. This method helps minimize the risk of exposing sensitive information since credentials are created for specific sessions and expire afterward. This is a big step up from static secrets, which can be leaked or mismanaged.

Traditional methods of managing secrets, like static secrets, require rotation and manual upkeep, which introduces human errors and increases the risk of leakage. For instance, it’s not uncommon for someone to accidentally commit a secret into a code repository. Removing such secrets from git history can be challenging. In addition, no matter how secure your static secret store is, there is nothing stopping the user from writing down the secret on a post-it note.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727529395565/2a176b16-3e59-4eb8-884d-203c11790454.png align="center")

### What You’ll Need

Before we begin, make sure you have the following tools and accounts ready:

* **HashiCorp Cloud Platform (HCP)**
    
* **Terraform CLI** (for infrastructure management)
    
* **Vault CLI** (for secret management)
    
* **AWS Account** (to use DocumentDB)
    
* **Docker** (to run containers)
    

### Steps Overview

We’ll break down the process into five easy-to-follow steps:

1. Deploy the AWS network and HCP Vault.
    
2. Set up DocumentDB with a bastion host.
    
3. Insert sample data into DocumentDB.
    
4. Configure Vault to generate dynamic credentials.
    
5. Test and access DocumentDB using the new credentials.
    
6. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727529445312/97e91adf-f27e-46ad-bf9c-f3be8bf9160a.png align="left")
    

Let’s get started!

## Step 1: Deploy Networking and HCP Vault

### 1.1 Update Terraform Configuration

We will use Terraform to set up our infrastructure. First, we need to configure our Terraform settings.

1. **Create a new directory** for your Terraform files.
    
2. Create a file named [`terraform.tf`](http://terraform.tf) and add the following:
    
    ```plaintext
    cloud {
      organization = "your-tfc-org-name"
      workspaces {
        name = "aws-network-hcp-vault"
      }
    }
    ```
    

Replace `"your-tfc-org-name"` with your actual organization name in Terraform Cloud.

### 1.2 Deploy Infrastructure

Next, we will set up our AWS resources.

1. **Open a terminal** and set your AWS credentials:
    
    ```bash
    export AWS_ACCESS_KEY_ID=<your-aws-key-id>
    export AWS_SECRET_ACCESS_KEY=<your-aws-access-key>
    ```
    
2. Navigate to your Terraform directory and run:
    
    ```bash
    terraform init
    terraform plan
    terraform apply --auto-approve
    ```
    

This will create the necessary networking resources in AWS.

### 1.3 Set Up VPC Peering

Go to your HCP Vault console to check that the VPC peering connection has been established. This allows Vault and DocumentDB to communicate securely.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727529550804/dbd0919f-b6c2-4674-8711-e243427522b2.png align="left")

For manual steps, see [HashiCorp developer docs for the HVN Quick Peering guide](https://developer.hashicorp.com/vault/tutorials/cloud-ops/amazon-peering-hcp?in=vault%2Fcloud-ops)

[⚠️ During the peering process, make sure you select th](https://developer.hashicorp.com/vault/tutorials/cloud-ops/amazon-peering-hcp?in=vault%2Fcloud-ops)e correct region where y[our AWS VPC is deployed.](https://developer.hashicorp.com/vault/tutorials/cloud-ops/amazon-peering-hcp?in=vault%2Fcloud-ops)

## [Step 2: Deploy DocumentDB wi](https://developer.hashicorp.com/vault/tutorials/cloud-ops/amazon-peering-hcp?in=vault%2Fcloud-ops)th a Bastion Host

### 2.1 Generate SSH Key Pair

To access DocumentDB securely, we’ll create an SSH key pair.

1. Run this command in your terminal:
    
    ```bash
    ssh-keygen -t rsa -b 4096
    ```
    
2. Press "Enter" to accept the default file location.
    

### 2.2 Update Terraform for DocumentDB

1. In a new [`terraform.tf`](http://terraform.tf) file for DocumentDB, add:
    
    ```plaintext
    cloud {
      organization = "your-tfc-org-name"
      workspaces {
        name = "aws-documentdb"
      }
    }
    ```
    

### 2.3 Deploy DocumentDB

Run the following commands to create your DocumentDB instance:

```bash
terraform init
terraform plan
terraform apply --auto-approve
```

### 2.4 Get the SSH Command

After deployment, note the SSH command provided in the Terraform output. It will look like this:

```plaintext
ssh_command = "ssh -L 27017:<your-docdb-cluster-address>:27017 ubuntu@<bastion-public-ip> -i ~/.ssh/id_rsa"
```

## Step 3: Insert Sample Records into DocumentDB

### 3.1 Establish SSH Tunnel

Use the SSH command from the previous step to connect to the bastion host:

```bash
ssh -L 27017:<your-docdb-cluster-address>:27017 ubuntu@<bastion-public-ip> -i ~/.ssh/id_rsa
```

### 3.2 Connect to DocumentDB

Now, we’ll connect to DocumentDB using the `mongosh` command:

```bash
mongosh "mongodb://<username>:<password>@<your-docdb-cluster-address>:27017/?ssl=true&retryWrites=false"
```

### 3.3 Create a Database and Insert Data

Once connected, create a new database and insert some sample data:

```javascript
use testdb
db.createCollection('collaboration')
db.collaboration.insertOne({'partners':'HashiCorp & AWS'})
```

## Step 4: Configure Vault for Dynamic Secrets

### 4.1 Build Docker Image

Now, we’ll set up Vault to manage our dynamic secrets.

1. Navigate to the Vault configuration directory and create a Docker image:
    
    ```bash
    cd vault-config/
    docker build -t db-configure-vault:latest .
    ```
    

### 4.2 Export Vault Environment Variables

Set your Vault environment variables:

```bash
export VAULT_ADDR=<vault-cluster-address>
export VAULT_TOKEN=<your-vault-token>
export DB_CLUSTER_ADDR=<your-docdb-cluster-address>
```

### 4.3 Run the Docker Container

Run the container to configure Vault:

```bash
docker run --name db-configure-vault --rm -e VAULT_ADDR -e VAULT_TOKEN -e DB_CLUSTER_ADDR db-configure-vault:latest
```

## Step 5: Test Your New Credentials

### 5.1 Generate Dynamic Credentials

Request dynamic credentials from Vault:

```bash
vault read database/creds/docdb-read-only-role
```

### 5.2 Access DocumentDB with New Credentials

Use the new credentials to log in to DocumentDB:

```bash
mongosh "mongodb://<new-username>:<new-password>@<your-docdb-cluster-address>:27017/admin?ssl=true&retryWrites=false"
```

### 5.3 Verify User and Data

Check that you can access your data:

```javascript
db.getUsers()
use testdb
db.collaboration.find()
```

## Clean Up

Once you're done testing, don’t forget to clean up your resources to avoid any charges:

```bash
cd terraform/documentdb/
terraform destroy --auto-approve
cd ../network-vault/
terraform destroy --auto-approve
```

## Conclusion

Congratulations! You’ve successfully set up AWS DocumentDB using dynamic secrets from HashiCorp Vault. This approach not only secures your database credentials but also ensures they are only valid when needed.

### Summary

We deployed HCP Vault and DocumentDB in AWS and saw that we can leverage the database secrets engine to generate just-in-time credentials. In addition, we further restricted the permissions tied to the credentials to only allow read-only access. Following the principle of least privilege, Vault can help ensure applications can only perform the allowed actions and access the data when needed.

### Clean Up

The infrastructure we deployed will cost money if we leave them running, so make sure to remove the resources!

* Exit out of any SSH and database connections
    
* Run terraform destroy from each of the following directories in this order: terraform/documentdb/, terraform/network-vault/
    
* Note: deleting the DocumentDB cluster may take ~10 minutes depending on cluster size
    

### Additional Resources

* [HashiCorp Vault Documentation](https://www.vaultproject.io/docs)
    
* [AWS DocumentDB Documentation](https://docs.aws.amazon.com/documentdb/latest/developerguide/what-is.html)
    
* [Terraform Documentation](https://www.terraform.io/docs/index.html)
    

Thank You For Reading My Blog! And Dont Forget To Share it, Hope It Will Help Someone Also!