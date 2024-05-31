
# Working with terraform on google cloud

As a prerequisite, create project and initialise cloudshell.

We will use terraform to create a network, firewall rules and vms.

Check terraform version.

#
    terraform --version

Create a folder for your terraform code
#
    mkdir tfinfra

Create file called provider.tf within the folder and paste: View the reference documentation [here](https://registry.terraform.io/providers/hashicorp/google/latest/docs)

#
    provider "google" {}

4. Initialise the terraform folder by:
#
    terraform init

5. Create file `mynetwork.tf` and paste the basic code for any resource: The reference for creating google vpc networks is in the [documentation](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_network)

```
# Create the mynetwork network
resource [RESOURCE_TYPE] "mynetwork" {
name = [RESOURCE_NAME]
# RESOURCE properties go here
}
```
    

6. To define a firewall rule to allow HTTP, SSH, RDP, and ICMP traffic on mynetwork, add the following base code to mynetwork.tf

```
    # Add a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on mynetwork
resource [RESOURCE_TYPE] "mynetwork-allow-http-ssh-rdp-icmp" {
name = [RESOURCE_NAME]
# RESOURCE properties go here
}
```

7. To create modules for reusability, create a new folder called `instances` and create `main.tf` and `variable.tf` files within the folder.

Add the following code to main.tf

```
resource "google_compute_instance" "vm_instance" {
  name = "${var.instance_name}"
  # RESOURCE properties go here
  zone         = "${var.instance_zone}"
  machine_type = "${var.instance_type}"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
      }
  }
  network_interface {
    network = "${var.instance_network}"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
    }
  }
}
```
    

8. 9Add the following code to variables.tf
```
variable "instance_name" {}
variable "instance_zone" {}
variable "instance_type" {
  default = "e2-micro"
  }
variable "instance_network" {}

```
    
9. Further add the following code to mynetwork.tf

```

# Create the mynet-us-vm instance
module "mynet-us-vm" {
  source           = "./instance"
  instance_name    = "mynet-us-vm"
  instance_zone    = "us-east1-d"
  instance_network = google_compute_network.mynetwork.self_link
}

# Create the mynet-eu-vm" instance
module "mynet-eu-vm" {
  source           = "./instance"
  instance_name    = "mynet-eu-vm"
  instance_zone    = "europe-west1-c"
  instance_network = google_compute_network.mynetwork.self_link
}
```

mynetwork.tf should resmble the following:
```
# Create the mynetwork network
resource "google_compute_network" "mynetwork" {
name = "mynetwork"
# RESOURCE properties go here
auto_create_subnetworks = "true"
}
# Add a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on mynetwork
resource "google_compute_firewall" "mynetwork-allow-http-ssh-rdp-icmp" {
name = "mynetwork-allow-http-ssh-rdp-icmp"
# RESOURCE properties go here
network = google_compute_network.mynetwork.self_link
allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
    }
allow {
    protocol = "icmp"
    }
source_ranges = ["0.0.0.0/0"]
}
# Create the mynet-us-vm instance
module "mynet-us-vm" {
  source           = "./instance"
  instance_name    = "mynet-us-vm"
  instance_zone    = "us-east1-d"
  instance_network = google_compute_network.mynetwork.self_link
}
# Create the mynet-eu-vm" instance
module "mynet-eu-vm" {
  source           = "./instance"
  instance_name    = "mynet-eu-vm"
  instance_zone    = "europe-west1-c"
  instance_network = google_compute_network.mynetwork.self_link
}
```


10. Format the code

#
    terraform fmt

11. Initialise, plan and apply

#
    terraform init

#
    terraform plan

#
    terraform apply


    

