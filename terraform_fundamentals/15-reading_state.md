# Lab: State

Duration: 10 minutes

This lab demonstrates how to read state from another Terraform project. It uses the simplest possible example: a project on the same local disk.

- Task 1:  Create a Terraform configuration that defines an output
- Task 2:  Read an output value from that project's state

## Task 1: Create a Terraform configuration that defines an output

For this task, you'll create two Terraform configurations in two separate directories. One will read from the other (using state files).

### Step 1.1

In this step, you'll create a Terraform project on disk that does nothing but emit an output. It should emit `public_ip` which can be a hard-coded value (for simplicity).

The project should consist of a single file which can be named something like `primary/main.tf`.

```bash
mkdir -p ~/workstation/terraform/azure/read_state_lab/primary && cd $_
```

```bash
touch main.tf
```

The contents of `main.tf` are a single output for `public_ip`. This is the entire contents of the file.

```hcl
# primary/main.tf
output "public_ip" {
  value = "8.8.8.8"
}
```

### Step 1.2

Generate a state file for the project. Within that project, run `terraform init` and `apply`. You should see a `terraform.tfstate` file after running these commands.

Run the standard `terraform` commands within the `primary` project.

```bash
terraform init
```

```bash
terraform apply
```

## Task 2: Read an output value from that project's state

### Step 2.1

Create a new Terraform configuration that uses a data source to read the configuration from the `primary` project.

Create a second directory named `secondary`.

```bash
mkdir ~/workstation/terraform/azure/read_state_lab/secondary && cd $_
```

```bash
touch main.tf
```

Define a `terraform_remote_state` data source that uses a `local` backend which points to the primary project.

```hcl
# secondary/main.tf
# Read state from another Terraform config’s state
data "terraform_remote_state" "primary" {
  backend = "local"
  config = {
    path = "../primary/terraform.tfstate"
  }
}
```

Initialize the secondary project with `init`.

```bash
terraform init
```

### Step 2.2

Declare the `public_ip` as an `output`.

Within `/workstation/terraform/azure/read_state_lab/secondary/main.tf`, define an output whose value is the `public_ip` from the data source you just defined.

```hcl
output "primary_public_ip" {
  value = data.terraform_remote_state.primary.outputs.public_ip
}
```

Finally, run `apply`. You should see the IP address you defined in the `primary` configuration.

```bash
terraform apply
```

```bash
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

primary_public_ip = 8.8.8.8
```
