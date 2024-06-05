# Build Infrastructure with Terraform on Google Cloud
## Terraform Fundamentals [GSP156]
### What is Terraform?
- 

### Verifying Terraform installation
- 

### Build Ingrastructure
- 

### Test your understanding
- 


## Infrastructure as Code with Terraform [GSP750]
### Build Infrastructure
-

### Change Infrastructure
-

### Create resource dependencies
-

### Provision Infrastructure
-


## Interact with Terraform Modules [GSP751]
### What is a Terraform module?
- 

### Use modules from the Registry
- 

### Build a module
- 


## Managing Terraform State [GSP752]
### Purpose of Terraform state
- 

### Working with backends
- 

### Import Terraform configuration
- 

### Limitations and other considerations
- 

- 

## Build Infrastructure with Terraform on Google Cloud: Challenge Lab [GSP345]
Challenge scenario
You are a cloud engineer intern for a new startup. For your first project, your new boss has tasked you with creating infrastructure in a quick and efficient manner and generating a mechanism to keep track of it for future reference and changes. You have been directed to use Terraform to complete the project.

For this project, you will use Terraform to create, deploy, and keep track of infrastructure on the startup's preferred provider, Google Cloud. You will also need to import some mismanaged instances into your configuration and fix them.

In this lab, you will use Terraform to import and create multiple VM instances, a VPC network with two subnetworks, and a firewall rule for the VPC to allow connections between the two instances. You will also create a Cloud Storage bucket to host your remote backend.

### Create the configuration files
- 

### Import infrastructure
- 

### Configure a remote backend
- 

### Modify and update infrastructure
- 

### Destroy resources
- 

### Use a module from the Registry
- 

### Configure a firewall
- 




[SOLUTION](https://github.com/quiccklabs/Labs_solutions/blob/6919838bdee340ac97f3d308d341758c1468806e/Build%20Infrastructure%20with%20Terraform%20on%20Google%20Cloud%20Challenge%20Lab/quicklab345.sh)

gcloud auth list

export REGION="${ZONE%-*}"

export PROJECT_ID=$DEVSHELL_PROJECT_ID

instances_output=$(gcloud compute instances list --format="value(id)")

# Read instance IDs into variables
IFS=$'\n' read -r -d '' instance_id_1 instance_id_2 <<< "$instances_output"

# Output instance IDs with custom name

export INSTANCE_ID_1=$instance_id_1

export INSTANCE_ID_2=$instance_id_2


touch main.tf
touch variables.tf
mkdir modules
cd modules
mkdir instances
cd instances
touch instances.tf
touch outputs.tf
touch variables.tf
cd ..
mkdir storage
cd storage
touch storage.tf
touch outputs.tf
touch variables.tf
cd


cat > variables.tf <<EOF

variable "region" {
 default = "$REGION"
}

variable "zone" {
 default = "$ZONE"
}

variable "project_id" {
 default = "$PROJECT_ID"
}
EOF




cat > main.tf <<EOF

terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "4.53.0"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region

  zone        = var.zone
}

module "instances" {

  source     = "./modules/instances"

}

EOF

terraform init 

cd modules/instances/

cat > instances.tf <<EOF

resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "n1-standard-1"
  zone         = "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "n1-standard-1"
  zone         = "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}
EOF

cd ~


terraform import module.instances.google_compute_instance.tf-instance-1 $INSTANCE_ID_1
terraform import module.instances.google_compute_instance.tf-instance-2 $INSTANCE_ID_2

terraform plan
terraform apply --auto-approve

cd modules/storage/

cat > storage.tf <<EOF

resource "google_storage_bucket" "storage-bucket" {
  name          = "$BUCKET_NAME"
  location      = "US"
  force_destroy = true
  uniform_bucket_level_access = true
}
EOF

cd ~


cat > main.tf <<EOF

terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "4.53.0"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region

  zone        = var.zone
}

module "instances" {

  source     = "./modules/instances"

}

module "storage" {
  source     = "./modules/storage"
}
EOF

terraform init
terraform apply --auto-approve

cat > main.tf <<EOF

terraform {
  backend "gcs" {
    bucket  = "$BUCKET_NAME"
 prefix  = "terraform/state"
  }
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "4.53.0"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region

  zone        = var.zone
}

module "instances" {

  source     = "./modules/instances"

}

module "storage" {
  source     = "./modules/storage"
}
EOF


echo "yes" | terraform init



cd modules/instances/

cat > instances.tf <<EOF

resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "e2-standard-2"
  zone         = "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "e2-standard-2"
  zone         = "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

resource "google_compute_instance" "$INSTANCE_NAME" {
  name         = "$INSTANCE_NAME"
  machine_type = "e2-standard-2"
  zone         = "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}
EOF
cd ~

terraform init

terraform apply --auto-approve


terraform taint module.instances.google_compute_instance.$INSTANCE_NAME

terraform plan
terraform apply --auto-approve

cd modules/instances/

cat > instances.tf <<EOF

resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "e2-standard-2"
  zone         = "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "e2-standard-2"
  zone         = "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}
EOF

cd ~

terraform apply --auto-approve


+++
cat > main.tf <<EOF

terraform {
  backend "gcs" {
    bucket  = "$BUCKET_NAME"
 prefix  = "terraform/state"
  }
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "4.53.0"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region

  zone        = var.zone
}

module "instances" {

  source     = "./modules/instances"

}

module "storage" {
  source     = "./modules/storage"
}

module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> 6.0.0"

    project_id   = "$PROJECT_ID"
    network_name = "$VPC_NAME"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = "$REGION"
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = "$REGION"
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "This subnet has a description"
        },
    ]
}

EOF

terraform init
terraform apply --auto-approve


cd modules/instances/

cat > instances.tf <<EOF

resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "e2-standard-2"
  zone         = "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "$VPC_NAME"
     subnetwork = "subnet-01"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "e2-standard-2"
  zone         = "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "$VPC_NAME"
     subnetwork = "subnet-02"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}


module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> 6.0.0"

    project_id   = "$PROJECT_ID"
    network_name = "$VPC_NAME"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = "$REGION"
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = "$REGION"
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "Please like share & subscribe to quicklab"
        },
    ]
}

EOF

cd ~

terraform init
terraform apply --auto-approve


cat > main.tf <<EOF

terraform {
  backend "gcs" {
    bucket  = "$BUCKET_NAME"
 prefix  = "terraform/state"
  }
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "4.53.0"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region

  zone        = var.zone
}

module "instances" {

  source     = "./modules/instances"

}

module "storage" {
  source     = "./modules/storage"
}

module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> 6.0.0"

    project_id   = "$PROJECT_ID"
    network_name = "$VPC_NAME"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = "$REGION"
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = "$REGION"
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "Please like share & subscribe to quicklab"
        },
    ]
}



resource "google_compute_firewall" "tf-firewall"{
  name    = "tf-firewall"
 network = "projects/$PROJECT_ID/global/networks/$VPC_NAME"

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_tags = ["web"]
  source_ranges = ["0.0.0.0/0"]
}

EOF

terraform init
terraform apply --auto-approve

