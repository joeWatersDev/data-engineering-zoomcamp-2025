# GCP AND TERRAFORM

Simply, Terraform is infrastructure as code. It allows you to define/create infrastructure via a text/config file. Some advantages include:

- Ease of tracking infrastructure - all services listed in a file
- Ease of collaboration - multiple people can modify the infrastructure in highly visible ways by modifying the text file
- Reproducibility - just reuse the terraform file with some updated parameters

Terraform runs on your local machine, but takes advantage of infrastructure provided by a platform, including Google Cloud Platform (what we will be using), AWS, Kubernetes, Microsoft Azure, etc.

Communication with the platform/service is done via a "provider". Some code written by users or the org that facilitates your terraform connection to the platform.

Key Terraform commands:

- Init: Gets the needed providers
- Plan: What is being done/created
- Apply: Run what's in the tf files
- Destroy: Remove all infrastructure defined in tf files