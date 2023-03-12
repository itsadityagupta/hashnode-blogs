---
title: "Storage Transfer Service in GCP"
datePublished: Sun Mar 12 2023 08:01:57 GMT+0000 (Coordinated Universal Time)
cuid: clf53xknx000e09me41o7hb3s
slug: storage-transfer-service-in-gcp
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1678349940643/a81553b7-faad-4dcb-9b9c-cefd3ceb7ce7.png
tags: terraform, gcp, data-engineering

---

## Introduction

The Storage Transfer Service is a **data migration service** provided by Google Cloud Platform (GCP) that allows users to move large amounts of data between on-premises data centres, cloud storage solutions, and other cloud providers.

The Storage Transfer Service can transfer data from sources such as Amazon S3, Microsoft Azure, and other third-party storage providers, as well as from file servers and other on-premises storage systems. Users can set up transfers that are one-time or recurring, with customizable scheduling and filtering options.

*The service supports various transfer options, including parallel transfer and resumable transfer, which helps to ensure data integrity and efficient data transfer. Additionally, it provides features like data validation, notification, and logging that enable users to monitor the transfer process and detect issues.*

The Storage Transfer Service can be managed using the GCP Console or through APIs, and it is charged based on the amount of data transferred and the number of operations performed.

---

In this blog post, I'll guide you through the process of creating a storage transfer service using Terraform to transfer data from one Google Cloud Storage (GCS) bucket to another. To follow along with this tutorial, you'll need to have the following prerequisites in place:

1. Terraform installed on your local machine
    
2. A Google Cloud Platform (GCP) project created in the Google Cloud Console
    
3. A GCP service account with a JSON private key file and the necessary permissions
    

If you haven't created a service account yet, check out my previous blog post on how to create one.

%[https://itsadityagupta.hashnode.dev/setting-up-a-gcp-service-account] 

## Assumptions

I have a Google Cloud Storage (GCS) bucket named "**gcs\_capstone\_dezoomcamp**" that contains approximately **9 gigabytes** of data. My goal is to transfer all the files from this bucket to the "data" folder within a **new** bucket called "**destination**".

> **Note:** The source bucket is already created whereas the destination bucket has to be created.

## Initializing a Terraform Project

1. Create a new directory for your Terraform project and create a new file named "**main.tf**". In the "**main.tf**" file, define the provider by specifying "google" as the provider and your project credentials.
    

%[https://gist.github.com/Aditya-Gupta1/4f30855c9a70477d662fe5a4325e81db] 

> Note: If the environment variable `GOOGLE_APPLICATION_CREDENTIALS` is set to the path of the service account key file, then the `credentials` fields is not necessary.

Run the `terraform init` command to initialize the project.

## Create the destination GCS bucket

Use the "**google\_storage\_bucket**" resource to define the destination bucket. Specify the "name" parameter to set the name of the bucket (globally unique), and the "location" parameter to set the location of the bucket.

%[https://gist.github.com/Aditya-Gupta1/8a785b0c503bf07f4aab7a1b6e888ae1] 

## Grant permissions to GCS buckets

Storage Transfer Service uses a *default service account* for data transfer which doesn't have required accesses by default. Hence, we have to configure access to the source GCS bucket and the destination GCS bucket before creating a data transfer job.

### Roles for source GCS bucket

1. One of:
    
    * **Storage Object Viewer** (`roles/storage.objectViewer`) if the transfer is to another Cloud Storage bucket.
        
    * **Storage Object Creator** (`roles/storage.objectCreator`) if the transfer is to a file system.
        
2. Plus one of the:
    
    * **Storage Legacy Bucket Writer** (`roles/storage.legacyBucketWriter`) if the object deletes permission is required.
        
    * **Storage Legacy Bucket Reader** (`roles/storage.legacyBucketReader`) if the object deletes permission is not required.
        

### Assigning roles to the source GCS bucket

To assign roles and permissions to the storage transfer service, it's important to first identify the default service account being used. This can be achieved by utilizing the "**google\_storage\_transfer\_project\_service\_account**" data block.

Once the service account has been identified, it can be granted the necessary roles and added as a member to the relevant Google Cloud Storage (GCS) bucket. Let's look at the terraform code to assign roles to the source GCS bucket:

%[https://gist.github.com/Aditya-Gupta1/a49dd76be109f5c242b285fc3641c792] 

*Note that the* `legacyBucketWriter` *role is not assigned to the source bucket because, in the data transfer job, I'll not delete the source files.*

### Roles for destination GCS bucket

* **Storage Legacy Bucket Writer** (`roles/storage.legacyBucketWriter`)
    

### Assigning roles to the destination GCS bucket

%[https://gist.github.com/Aditya-Gupta1/8b6fc66fd3ff91f76befa86e254fcf93] 

## Define the data transfer job

Use the "**google\_storage\_transfer\_job**" resource to define the data transfer job. Specify the source and destination bucket names as well as the desired transfer options.

%[https://gist.github.com/Aditya-Gupta1/536facac85cf99eea1398f3ecb90ca64] 

## Overview

Here's a complete **"main.tf"** file:

%[https://gist.github.com/Aditya-Gupta1/3db7dedf4e85dd9bcc1d44632676441f] 

Now, **run the command** `terraform apply` and enter "yes" when prompted. This will perform the following:

* Create a destination bucket.
    
* Assign roles to the default service account for the source and the destination bucket.
    
* Create a data transfer job ***with a schedule as defined***.
    

> Make sure to run `terraform destroy` to destroy all the resources.

## Conclusion

By following the steps outlined in this blog post, you can easily define a storage transfer job and assign the necessary permissions to the source and destination buckets. This allows you to transfer data between buckets or even between different cloud storage providers.

More information about this can be found in the following blogs:

* [https://cloud.google.com/storage-transfer-service](https://cloud.google.com/storage-transfer-service)
    
* [https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/storage\_transfer\_job](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/storage_transfer_job)
    
* [https://cloud.google.com/storage-transfer/docs/sources-and-sinks](https://cloud.google.com/storage-transfer/docs/sources-and-sinks)
    

I hope you found this blog post helpful and informative. I'd love to hear your thoughts on the topic or any issues you've had following along. Did I miss anything important?

Please leave a comment below and let me know what you think. Your feedback is valuable and can help me improve my writing and provide additional insights for other readers. Thank you for taking the time to read this post!

%%[linkedin-follow]