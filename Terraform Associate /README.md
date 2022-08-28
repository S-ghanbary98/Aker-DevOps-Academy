# Terraform Associate

- In this directory, projects involving terraform will be displayed here along with a study guide on the Terraform Associate exam.

### What is Terraform?

- An open source IaC software tool that allows us to provision, manage, and change infrastructure through code, specifically HCL (Hashicorp Congiuration Language).

### Terraform Workflow

- 3 phases; write - plan - apply
- write - involves the starting off with version control software ie. github/gitlab
- plan - plan, review changes code will make to environment
- Apply - deploy changes to infractructure 


### Terraform Commands

```
# to initialize a working directory where terraform code will be stored.
# Also to setup the backend where state files are stored for terraform to track resources.

terraform init
```

```
# part of the plan phase, a read only command to then show a 'plan' of the execution/deployment.
# Allows users to review action plan, no changes are made and credentials/authorizastion is required.

terraform plan
```

```
# part of the deploy phase, an execution command to carry out statements in the terraform file.
# updates the state file, thats used to track deployments.

terraform apply
```

```
# part of the deploy phase, an exxecution command to carry out statements in the terraform file.
# Looks at recoreded state file created in the deployment, to then destroy that infrastructure and all resources.
# a non reversible command so backups should be taken.

terraform destroy
```


### Terraform Code

- firstly you start of the code with the provder you are using, e.g AWS, GCP. Then you move on to provision the resources.
- Terraform executes file with `.tf` extension.
- Terraform looks for providers in terraform providers registry, but providers can also be sources locally within terraform code.

#### Provider

- Terraform finds and installs neccessary packages for the providers when carrying out the `terraform init` command.

```
provider "<PROVIDER NAME>" {
    <CONFIGURATION PARAMETERS FOR PROVIDER>
    }
```

- AWS Example

```
provider "aws" {
    region = "us-east-1"
    version = "3.08.22"
}
```


#### Resources

- create a resource.

```
resource "<RESOURCES PROVIDED BY PROVIDER>" "<CUSTOM NAME TAG>"{
    "<RESOURCE CONFIG>"
}
```

- AWS example

```
resource "aws_instance" "my web server"{
    ami = "ami-1463434"
    instance_type = "t2.micro"
}
```
- The resource address then becomes resource.aws_insatnce.mywebserver

#### Data

- for fetching and tracking an existing resource.

```
data "<RESOURCES PROVIDED BY PROVIDER>" "<CUSTOM NAME TAG>"{
    "<DATA SOURCE ARGUMENTS>"
}
```

- AWS example

```
resource "aws_instance" "my vm"{
    instance_id = "i-134765j5j52"
}
```

- The resource address then becomes data.aws_insatnce.my_vm


### Terraform Resource Tracking: State file.

- The mechanism for terraform to know what has been deployed, for terraforms fucntionality. e.g to see whether it is deploying infrastructure, changing infrastructure or destroying it.
- Contains all metadata about deployemnt and resources. Helps calculate deployment delta and create new deployment plans.
- `terraform.tfstate` default name of file.
