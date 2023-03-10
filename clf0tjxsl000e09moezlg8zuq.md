---
title: "Setting up a GCP Service account"
datePublished: Thu Mar 09 2023 08:00:20 GMT+0000 (Coordinated Universal Time)
cuid: clf0tjxsl000e09moezlg8zuq
slug: setting-up-a-gcp-service-account
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1678348699724/98a91cfa-9646-4577-98cc-062f47ac2bdd.png
tags: gcp

---

## Introduction

A service account in Google Cloud Platform (GCP) is a special type of account that allows applications running on GCP to interact with other GCP services, such as Google Cloud Storage, Google Cloud SQL, and Google Compute Engine.

Service accounts are typically used by applications that run on GCP, such as virtual machines, containers, and applications running on Google Kubernetes Engine. Using service accounts, applications can securely access GCP resources without requiring users to authenticate individually.

Each service account is associated with a set of credentials, which typically include a private key that can be used to authenticate the service account with other GCP services. Service accounts can also be granted specific permissions to access particular resources, making it easy to control access to GCP resources on a per-service basis.

In this blog post, I will guide you through the step-by-step process of creating a service account on the Google Cloud Platform (GCP). This account will be specifically created to be used for my future blog posts on GCP. By following this tutorial, you will be able to create a service account and obtain the necessary credentials to interact with other GCP services.

The instructions provided in this post will be concise and easy to follow, ensuring that you can complete the process quickly and efficiently. In case you need to create a service account in the future, you can refer back to this post for guidance. Let's get started!

## Creating a Service Account

1. Sign in to your google cloud console at [**https://console.cloud.google.com**](https://console.cloud.google.com).
    
2. From the dropdown at the top, select the project you want your service account to be associated with.
    
3. From the navigation menu on the left-hand side, select "**IAM & Admin**" and then select "**Service accounts**".
    
4. Click the "**Create service account**" button at the top of the page.
    
5. Enter a name for your service account and provide an optional description. Click "**Create and Continue**".
    
6. Under the "**Grant this service account access to project**", select the following roles to be assigned to the service account and click "**Continue**" (you can modify the roles as per your requirements):
    
7. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678346726486/c723ec2f-f813-48bd-b54b-3154e4d14e24.png align="center")
    
    Click "**Done**".
    

## Creating a Service Account Key

1. From the navigation menu on the left-hand side, select "**IAM & Admin**" and then select "**Service accounts**".
    
2. Locate the service account that you created and click on its name.
    
3. Click on the "**Keys**" tab and then click "**Add Key**".
    
4. Click on the "**Create new key**" button, select "**JSON**" as the Key Type and click "**Create**".
    
5. This will download the JSON key file to your local machine.
    

## Setting up the environment variable

One last thing is to create an environment variable `GOOGLE_APPLICATION_CREDENTIALS` and point it to the path of the downloaded key file.

## Conclusion

Now, you have a service account that can be used to interact with other GCP services and infrastructure management (through IaaC tools like Terraform etc.) as well.

To learn more about service accounts, you can go over the official documentation here: [https://cloud.google.com/iam/docs/service-account-overview](https://cloud.google.com/iam/docs/service-account-overview)

I hope you found this blog post helpful and informative. I'd love to hear your thoughts on the topic or any issues you've had following along. Did I miss anything important? Please leave a comment below and let me know what you think. Your feedback is valuable and can help me improve my writing and provide additional insights for other readers. Thank you for taking the time to read this post!

%%[linkedin-follow]