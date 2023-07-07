# terraform-with-github-action
Deploying Terraform Code with GitHub Action using Workload Identity Federation

Title: Deploying Resources on Google Cloud Platform (GCP) using Terraform with GitHub Action using Workload Identity Federation

Introduction:
Integrating Google Cloud Platform (GCP) with GitHub can streamline your development and deployment processes. By leveraging Workload Identity Federation, you can establish a secure connection between these platforms. In this blog post, we will guide you through the necessary steps to connect GCP with GitHub using Workload Identity Federation. Let's get started!

Please note that the following instructions assume you have already set up a GCP project and have the necessary permissions to create and manage resources.

Step 1: Enable necessary APIs
To begin, you need to enable the required APIs in your GCP project. These APIs include the Identity and Access Management (IAM) API, Cloud Resource Manager API, and Compute Engine API. Follow these steps to enable the APIs:

1. Go to the GCP Console: https://console.cloud.google.com/.
2. Open your project.
3. Click on the "Navigation Menu" button (three horizontal lines) in the upper-left corner and select "APIs & Services" > "Library" from the menu.
4. Search for the following APIs and enable them if they are not already enabled:
   - Identity and Access Management (IAM) API
   - Cloud Resource Manager API
   - Compute Engine API

Step 2: Set up a service account for Workload Identity Federation
Next, you need to create a service account and assign the necessary roles for Workload Identity Federation. Here's how to do it:

1. In the GCP Console, go to "IAM & Admin" > "Service Accounts".
2. Click on "Create Service Account".
3. Provide a name and description for the service account and click on "Create".
4. Assign the necessary roles to the service account, such as "Compute Instance Admin" and "Workload Identity User".
   - Note: The required roles may vary depending on your use case. Make sure to assign the appropriate roles based on your requirements.
5. Click on "Done".

Step 3: Set up Workload Identity Federation
In this step, you'll configure the Workload Identity Federation in GCP. Follow the instructions below:

1. In the GCP Console, go to "IAM & Admin" > "Workload Identity Federation".
2. Click on "Add Provider".
3. Provide a name and description for the provider.
4. Select "OpenID Connect (OIDC)" as the provider type.
5. Click on "Create".
6. In the "Configure Provider" tab, click on "Create Mapping".
7. Provide a name and description for the mapping.
8. Click on "Create".
9. Once the pool is created, you need to grant access to your service account:
   - Click on the pool you created.
   - Click on "Grant Access".
   - Select your service account name from the drop-down menu and save the changes.

Step 4: Set up GitHub
Now, let's configure the necessary secrets in GitHub repository settings:

1. Go to the GitHub repository you want to connect with GCP.
2. Navigate to "Settings" > "Secrets".
3. Click on "New repository secret".
4. Provide a name for the secret (e.g., `PROJECT_ID`, `PROJECT_REGION`, `PROJECT_ZONE`) and enter the GCP project ID, zone, and region as the value.
5. Repeat the above steps to create another secret with the name `SERVICE_ACCOUNT`, and paste the service account email you created in Step 2 as the value.
6. Repeat the above steps to create another secret with the name `POOL_URL`, and paste the Workload Identity Federation Pool URL you created in Step 3 as the value.

Step 5: Configure GitHub Actions for deployment
Now, let's set up GitHub Actions

to automatically deploy your Terraform code using Workload Identity Federation:

1. In your GitHub repository, go to the "Actions" tab.
2. Click on "Set up a workflow yourself" or "New workflow".
3. Replace the content of the generated YAML file with the following code:

```yaml
name: Deploy Terraform with Workload Identity Federation

on:
  push:
    branches:
      - main

jobs:
  terraform-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v1
        with:
          token_format: 'access_token'
          workload_identity_provider: ${{ secrets.POOL_URL }}
          service_account: ${{ secrets.SERVICE_ACCOUNT }}
          project_id: ${{ secrets.PROJECT_ID }}

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_version: 1.2.5

      - name: Terraform fmt
        id: fmt
        run: terraform fmt -check
        continue-on-error: true

      - name: Terraform Plan
        id: plan
        run: terraform plan

      - name: Terraform Plan Status
        if: steps.plan.outcome != 'success'
        run: exit 1

      - name: Terraform Apply
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: terraform apply -auto-approve -input=false
```

4. Save the file.

That's it! Whenever you push changes to the main branch of your GitHub repository, the GitHub Actions workflow will trigger and authenticate with GCP using the service account key. This workflow will deploy your Terraform code with the help of Workload Identity Federation, ensuring a secure connection between GCP and GitHub.

Terraform File Structure:
To deploy your infrastructure using Terraform, you'll need to create Terraform configuration files. Here's a typical file structure:

1. Create a file named `main.tf` in the root directory of your repository. This file contains the main Terraform configuration.

```
    resource "google_compute_instance" "example" {
      name         = "example-instance"
      machine_type = "n1-standard-1"
      zone         = var.gcp_zone
    
      # Other configurations for your compute instance
    }
    
```
2. Create a file named `provider.tf` in the same directory. This file defines the provider used in your Terraform configuration.
```
  provider "google" {
    project = var.project_id
    region  = var.gcp_region
}
```
3. Create a file named `variables.tf` in the same directory. This file defines the variables used in your Terraform configuration.

```hcl
variable "project_id" {
  type    = string
  default = ""
}

variable "gcp_region" {
  type    = string
  default = ""
}

variable "gcp_zone" {
  type    = string
  default = ""
}
```
Make sure to replace the default values in the variables with the corresponding values from your GitHub Secrets.

Conclusion:
By following the steps outlined in this blog post, you can successfully connect Google Cloud Platform (GCP) with GitHub using Workload Identity Federation. This integration enables seamless authentication and secure communication between the two platforms. With GitHub Actions and Terraform, you can automate infrastructure deployments and enhance your development workflow. Empower your team

with the capabilities of GCP and GitHub, accelerating your software development process.

Remember to customize the provided code and instructions according to your specific use case and requirements. Make sure to replace the placeholder values with your actual GCP project details and customize the Terraform configuration files to define your desired infrastructure.

With the power of Workload Identity Federation, GCP, GitHub, and Terraform, you can achieve efficient and secure infrastructure deployments. Happy integrating GCP with GitHub using Workload Identity Federation!

If you have any further questions or need additional assistance, feel free to ask.
