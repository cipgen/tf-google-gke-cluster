
# Google Kubernetes Engine (GKE) Cluster Terraform module

This module deploys a Kubernetes cluster on Google Cloud Platform (GCP) using the Google Kubernetes Engine (GKE) service. The GKE cluster is provisioned with a single node pool, and it comes with a generated Kubernetes configuration file (`kubeconfig`) that is stored locally.

## Usage

```terraform
provider "google" {
  # Configuration options
  project = var.GOOGLE_PROJECT
  region  = var.GOOGLE_REGION
}

resource "google_container_cluster" "this" {
  name     = var.GKE_CLUSTER_NAME
  location = var.GOOGLE_REGION

  initial_node_count       = 1
  remove_default_node_pool = true
}

resource "google_container_node_pool" "this" {
  name       = var.GKE_POOL_NAME
  project    = google_container_cluster.this.project
  cluster    = google_container_cluster.this.name
  location   = google_container_cluster.this.location
  node_count = var.GKE_NUM_NODES

  node_config {
    machine_type = var.GKE_MACHINE_TYPE
  }
}

module "gke_auth" {
  depends_on = [
    google_container_cluster.this
  ]
  source               = "terraform-google-modules/kubernetes-engine/google//modules/auth"
  version              = ">= 24.0.0"
  project_id           = var.GOOGLE_PROJECT
  cluster_name         = google_container_cluster.this.name
  location             = var.GOOGLE_REGION
}

resource "local_file" "kubeconfig" {
  content  = module.gke_auth.kubeconfig_raw
  filename = "${path.module}/kubeconfig"
}

output "kubeconfig" {
  value = "${path.module}/kubeconfig"
}
```

## Inputs

|       Name       |            Description           |  Type  |     Default     | Required |
|:----------------:|:--------------------------------:|:------:|:---------------:|:--------:|
| GOOGLE_PROJECT   | GCP project name                 | string | no              |    no    |
| GOOGLE_REGION    | GCP region name                  | string | "us-central1-c" |    no    |
| GKE_MACHINE_TYPE | GKE node machine type            | string | "g1-small"      |    no    |
| GKE_NUM_NODES    | Number of nodes in the node pool | number | 2               |    no    |

## Outputs
kubeconfig - Generated Kubernetes configuration file

## Requirements
This module requires Terraform 0.12 or later, and the following provider:

hashicorp/google 4.52.0

## License
This module is licensed under the MIT License. See the LICENSE file for details.

--------

# Branch  Mod7_t1 Infracost

**Steps for Terraform Repository Management and Infracost Integration**

1. **Forking the Git Repository:**
   - Begin by forking the repository with Terraform code: [https://github.com/den-vasyliev/tf-google-gke-cluster](https://github.com/den-vasyliev/tf-google-gke-cluster). This creates a copy of the repository in your own GitHub account, which you can work on and modify locally.

2. **Verifying Terraform Configuration:**
   - Using Google Cloud Shell, navigate to the repository and run `terraform init`, `terraform validate`, and `terraform plan` to check the Terraform configuration for errors and ensure that the plan is correct.

3. **Installing and Authenticating Infracost:**
   - Install Infracost in Google Cloud Shell:
     ```bash
     curl -fsSL https://raw.githubusercontent.com/infracost/infracost/master/scripts/install.sh | sh
     ```
   - Authenticate with Infracost by running `infracost auth login`.

4. **Setting up Main Branch Protection:**
   - Enable branch protection for the `main` branch to prevent accidental changes or unauthorized modifications.

5. **Configuring Infracost Integration:**
   - Add Infracost integration to the repository, following the instructions at [Infracost Documentation](https://www.infracost.io/docs/). This allows you to automatically calculate and control infrastructure costs when making changes to the repository.

6. **Testing Changes with a Pull Request:**
   - Create a pull request to introduce changes to the Terraform code, such as the number and type of nodes in the cluster. Test the changes to ensure everything works correctly.

