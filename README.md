Here's a detailed guide on what you can add to your README document to explain the project, steps, and how users can reproduce it. This README will help others understand your project, set up the environment, and deploy their own VM using Cloud Build and GitHub.

---

## GCP VM Deployment using Terraform and Cloud Build

### Overview

This project demonstrates how to automatically deploy a Virtual Machine (VM) on Google Cloud Platform (GCP) using Terraform, GitHub, and Cloud Build. The CI/CD pipeline is triggered whenever new changes are pushed to the GitHub repository, ensuring seamless and automated VM creation using GCP's Cloud Build service.

---

### Architecture

The project follows a **Continuous Deployment** model:

1. **Infrastructure as Code (IaC)** is written using Terraform.
2. **Cloud Build** is configured to automatically deploy a GCP VM whenever code is pushed to the GitHub repository.
3. **Terraform** creates and manages the VM instance in GCP based on the configuration files.

---

### Features

- **Automated Infrastructure Deployment** using Cloud Build.
- **Terraform as IaC**: Define and manage your infrastructure using simple Terraform configuration files.
- **GitHub Integration**: Trigger deployments via pushes to the GitHub repository.
- **IAM Roles**: Secure and control access to GCP resources using Cloud Build's service account.

---

### Prerequisites

Before proceeding, ensure you have:

- A **Google Cloud Platform (GCP)** account.
- A **GitHub** account with a repository set up for the project.
- **Google Cloud SDK** installed and configured on your local machine.
- **Terraform** installed (You can get it from [Terraform's official website](https://www.terraform.io/downloads)).

---

### Getting Started

#### 1. Clone the Repository

To get started, clone the repository to your local machine:

```bash
git clone https://github.com/your-username/gcp-vm-deployment.git
cd gcp-vm-deployment
```

#### 2. Set up Terraform Configuration

Inside the repository, we have the following key files:

- **main.tf**: Defines the VM instance configuration.
- **variables.tf**: Contains the input variables for project ID and region.
- **terraform.tfvars**: Specifies the values for the variables.
- **cloudbuild.yaml**: Defines the build steps executed by Cloud Build to deploy the infrastructure.

Example `main.tf` for deploying a VM:

```hcl
provider "google" {
  project = var.project_id
  region  = var.region
}

resource "google_compute_instance" "vm_instance" {
  name         = "demo-vm"
  machine_type = "f1-micro"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "default"
    access_config {}
  }
}
```

---

### Steps to Deploy

#### 1. Enable Required APIs on GCP

Ensure the following APIs are enabled in your GCP project:

- **Cloud Build API**
- **Compute Engine API**
- **IAM Service Account API**

You can enable them with the following command:

```bash
gcloud services enable cloudbuild.googleapis.com compute.googleapis.com iam.googleapis.com
```

#### 2. Set up GitHub and Cloud Build Integration

1. **Connect GitHub Repo**: In the GCP Console, navigate to **Cloud Build** → **Triggers** → **Create Trigger**.
   - Connect your GitHub repository to Cloud Build.
   - Set the trigger to fire on any changes pushed to the `main` branch.

2. **Cloud Build Permissions**: Ensure that Cloud Build’s service account has the following roles:
   - **Compute Admin**
   - **Service Account User**

   Grant permissions using:

   ```bash
   PROJECT_ID=$(gcloud config get-value project)
   CLOUD_BUILD_SA="${PROJECT_ID}@cloudbuild.gserviceaccount.com"

   gcloud projects add-iam-policy-binding $PROJECT_ID \
       --member serviceAccount:$CLOUD_BUILD_SA \
       --role roles/compute.admin

   gcloud projects add-iam-policy-binding $PROJECT_ID \
       --member serviceAccount:$CLOUD_BUILD_SA \
       --role roles/iam.serviceAccountUser
   ```

#### 3. Push Your Code

Push any changes or updates to your GitHub repository:

```bash
git add .
git commit -m "Updated Terraform files"
git push origin main
```

#### 4. Monitor the Build

- Go to **Cloud Build** in the GCP Console to view the build logs.
- Once the build is successful, you can check the **Compute Engine** section to see the deployed VM.

---

### Pipeline Explanation

The **cloudbuild.yaml** defines how the Terraform code is executed by Cloud Build.

```yaml
steps:
  - name: 'hashicorp/terraform:latest'
    entrypoint: 'sh'
    args:
      - '-c'
      - |
        terraform init
        terraform apply -auto-approve
```

- **terraform init**: Initializes Terraform, downloading provider dependencies.
- **terraform apply**: Applies the Terraform configuration, creating the GCP VM instance.

---

### Clean Up

To avoid incurring costs, you can clean up the resources by destroying the VM.

You can either:
- Manually destroy the resources using Terraform:

  ```bash
  terraform destroy -auto-approve
  ```

- Modify the `cloudbuild.yaml` to include a destroy step and trigger it through GitHub.

---

### Future Improvements

- Integrate other GCP services like Cloud Storage, Cloud SQL, etc.
- Automate more complex infrastructure deployment using Terraform modules.
- Implement secret management using **Google Secret Manager**.

---

### Troubleshooting

- **Permissions Issues**: Ensure that Cloud Build has the necessary permissions to create resources.
- **Build Failures**: Check the Cloud Build logs for detailed error messages.
- **Terraform Errors**: Validate your Terraform syntax using `terraform validate`.

---

### Resources

- [Google Cloud Build Documentation](https://cloud.google.com/build/docs)
- [Terraform GCP Provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs)
- [GitHub and GCP Integration](https://cloud.google.com/build/docs/automating-builds/create-github-app-trigger)

---
