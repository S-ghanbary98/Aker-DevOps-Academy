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

### Terraform State Command

- Terraform state maps real world resources to resources you map in your configuration.
- stored locally or remotely in `terraform.tfstate`.
- prior to modification, the state file is refreshed.
- resource dependancy is also tracked in the state file. e.g subnet must be configured first to then configure a VM in aws cloud.
- helps performance by caching resource attributes for subsequent use.

Terraform state command is for manipulating and reading the terraform state file, some scenarios this would be done is

- Advanced state management.
- Remove resource from state file so it's not tracked by terraform.
- List out tracked resources and their use via list sub commands.

```
#list all resources in terraform state file.

terraform state list

# Deleted resource from state file
terraform state rm

# show details of tracked resources in state file.
terraform state show
```

### Local and remote state storage

By default terraform is stored locally. Terraform does backup last knowns state file after a successful `terraform apply` as a fail safe. State can also be locked so that parallel executions don't coincide. State locking is activated locally by default but may not be for some cloud providers. Output sharing is also possible where state file contains env variables or set variables about the infrastructure for another team to then reference/use with their own terraform code.

- use the `backend` attrubute to configure the remote saved state.


### Terraform Modules

- A modules is a collection of terraform code files in a folder, and you reference the output of the code into other terraform projects.
- It groups together multiple resources.
- It makes code re-usable in other projects.
- Every terraform configuration has atleast one module ,`the root module`, which conists of code files in your main working directory.


Modules can be downloaded or reference from :

- Terraform public registry - terraform downloads them and stores them in your system. A hidden directory.
- A private registry
- Stored locally and reference via path.


Modules are reference using `module` block.

```
# other parameters in the module can be - count, for_each, providers, depends_on:set dependacies on module.

module "my-vpc-module"{
    source = "./modules/vpc"
    version = "0.0.13"
    region = var.region
}

#inside the block you can reference variables within for examle region could be referenced within the block as var.region

```

- Modules can take input and also give an output to then pass in your main code. e.g in `resource` block add example `subnet_Id = module.my-vpc-module.subnetID`
- module.<name of module>.<name of parameter>


### Terraform Built in Functions

- Terraform comes with pre-packaged functions to help transform and combine values, as user defined functions are not allowed.

```
# Join is an inbuilt function that combines the name of the variable to the word 'terraform', along with a seperator.

variable 'project-name' {
    type = string
    default = 'pod'
}

resource 'my_vpc' {
    cidr_block = '10.0.0.0/16'
    tags = join('-', ['terraform', var.project-name])
}
```

- `terraform console` provides an interactive console for evaluating expressions, and test built in functions.


### Terraform Type Contraints

- Type constraints control the type of variable value.
primitive - single type value - string, bumber, bool
complex - list, tuple, map, object

Complex can ve broken down to :

- Collection - multiple values of one primitive value - list(string) A list of strings
- Structural - Multiple values of differenc primitive types. set(type) 

```
variable "my_object" {
    type = object({
        name = string
        age = number
    })

}

```
- The `any` Contraint which is a placeholder for primitive types, yet to be decided. Actual value determined at runtime.

```
variable "my_data" {
    type = list(any)
    default = [1, 2, 4]
}
```

### Terraform Dynamic Blocks

- Dynamically contructs repeatable nested configurations blocks inside terraform resources.
- Expect a complex variable.
- Acts like a for loop, stops repeating code writing.

supported within following blocks
- Resource 
- data
- provider
- provisioner

### Terraform fmt, taint and import

#### Terraform fmt (format)

- formats terraform code for readability.
- Keeps code consistant.
- Safe to run whenever since only affects asthetics.
- Formats everythng with a .tf extenstion.

#### Terraform taint

- marks an existingt resourceto be destroyed then recreated.
- Modifies the state file so the next terraform apply destroys and recreates the resource.

`terraform taint <resource address>`

Uses include:
- when you want provisioners to run.
- replace misbehavng resources.
- mimic recreation for testing.


#### Terraform import 


- Maps existing resources not tracked by terraform to terraform using an ID.
- `ID` is dependant on the underlying vendor. e.g an EC2 ID will be its `instance ID`.
- Importing same resource to multiple terraform resources is not recommended.

`tarrform import <RESOURCE ADDRESS> <ID>`

Uses include:
- Use existing resources.
- not allowed to create new resources.
- Not in control of creation process in infrastructure.


#### Terraform Configuration Block

- A special block for conrolling terraform block.


```
terraform {
    required_version = ">=0.13.0"
    required_providers {
        aws = ">=3.0.0"

    }
}
```

### Terraform workspaces

- Simply Alternate state files in the same working drectory.
- Terraform start with a default workspace that cannot be deleted.
- Each workspace tracks a copy of the terraform state file in the working directory.
- `${terraform.workspace}`  to access current workspace.

```
#Create a new workspace
terraform workspace new <workspace NAME>

#Select a workspace
terraform workspace select <workspace NAME>

# list all workspaces and current workspace
terraform workspace list
```

uses:
- test parallel distinct copies of infrastrucure.
- modeled against braches in version control.
- good for sharing workspaces and collaboration between teams. Test terraform without altering default state file.

### Terraform Debugging

- `TF_LOG` is an env for enabling verbos logging in terraform , by default it will send logs to stderr(standard output error).
- It can be set to levels of, `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`.
- `TRACE` is the most verbose one and reliable one.
- Set `TF_LOG_PATH` to a text file to save log outputs. defualt value is disabled.


### Sentinel Benefits

- POLICY AS CODE
- Enforces policies on code. Add restrictions.
- Use sentinel language to stop maliciouse terraform code, that stops it before a `terraform apply` and after `tarraform plan`.
- human readable non-technical.


- Sandboxing - Stop dev user deploying to prod, guardrails for automation.
- Codification - easier understanding, and collab.
- version controlled.
- good fore security testing.

uses:
- Enforce CIS standats in aws.
- Restrict EC2 type used.
- Ensure SC groups dont allow port 22.
- Ensure all resources have tags.


### Terraform Vault

- A secrets management software
- provides short-lived creds, adn stores sensitvie data.
- Encrypts sensitive data in transit and at rest, and provides access using ACL.
- Provides IAM access to code as well as creds to carry out deployments.

- Long lived keys worn be required anymore e.g secret access keys, as all keys provided by vault are temporary.
- Inject secrets in terraform code.
- Fine grained ACL for access to vault.



### Terraform Registry

- A registry of terraform providers and modules.
- You can publish and share modules in the registry.
- Can be directly referenced in terraform code.

`https://registry.terraform.io/`


### Terraform Cloud workspaces

- All hosted in the terraform cloud.
- Can ntract with via api as well.
- Stores all old state files by defualt.
- Maintains a record of all activity.
- Trigger via api or github actions not just CLI.

Benefits

- Remote execution via api.
- Workspace based organisational model.
- Version control.
- Remote state management and cli integration.
- Private terraform module registry.
- Cost optimized and sentinel integration.

### Terraform OSS vs Cloud Workspace

- Stores alternate state files in the same working directory

### miscellaneous


- `terraform validate`: The terraform validate command validates the configuration files in a directory, referring only to the configuration and not accessing any remote services such as remote state, provider APIs.