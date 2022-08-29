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


### Terraform Variables and Outputs

#### Set Variable

- Declare a terraform variable via the reserved keyword `variable`. Info within such as `type` and `default` are optional config values.
- can make code more versatile.
- Can be specified in terraform file but best practice to place variables within `terraform.tfvars`
```
variable "<VARIABLE-NAME>" {
    description = "this is my var"
    type = string
    default = "Hello!"
}

#no decleration but variable can/must be assigned via an environmnnet variable for expression below or via cli.
# terraform apply -var myVar=1

variable "myVAR" {}

```
- Variable is referenced via `var.<VARIABLE-NAME>`, example for one above `var.myVar`.

#### Variable Validation

- Variable validation allows you to set a criteria for the allowed values for a variable.
- Terraform version 0.13 and above, but was expiremental for version 0.12.
- `sensitive` parameter keeps it a secret when terraform is executing. A boolean valeu with a default of false.

```
variable "<my-Var>" {
    description = "this is my var"
    type = string
    default = "Hello!"
    sensitive - true

    validation {
      condition   = length(var.my-Var) > 4
      error-message = "String must be more than 4 characters"
    }

}
```

#### Constraints

Base Types:
- string
- numbers 
- bool

Complex Types:
- lists, sets, map, object, tuple


```
# list of strings variable

variable "<my-Var>" {
    type = list(string)
    default = ['us-east-1']
}
```

```
# list of objects variable
# highest precedence is given to environment varables instead of the variables set in the terraform files.

variable "<my-Var>" {
    type = list(objects({
        internal = number
        external = number
        protocol = string
        }))

    default = [{
        internal = 8300
        external = 8300
        protocol = 'tcp'
    }
  ]
}
```

#### Outputs

- Reserved keyword output.
- Outputs are shown in the CLI after a terraform apply.
- Just give a return value in console for user.

```
output "Instance_IP" {
    description = "vm private IP"
    value = aws_instance.my-var.private_ip

}
```


### Terraform Provisioners

- Provisioners give a way to execute custome script and command through terraform resources.
- Can be run locally or on spun up VM in the cloud after a deployment.
- Each individuale resource like an ec2within the terraform code can have its own `provisioner` defining the connection method 'ssh/winRM' to then action the scripts.

- There are two types of provisioners; `create-time` and `destroy-time` which you can set to run when resource is being created or destroyed. create is the default.

<strong>Terraform cannot track provisioners in state files, best practice is to use another method with provisioners as a safety net incase it can't be done.</strong>

- Provisioners are recommended when you want to invoke actions not covered by terraform decleration model.
- Return code must be zero. Execute in sequence.

```
resource "null_resource" "dummy_resource" {
    provision "local-exec" {
        command = "echo '0' > status.txt"
    }

    provision "local-exec" {
        when = destroy
        command = "echo '1' > status.txt"
    }
}

```