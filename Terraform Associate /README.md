# Terraform Associate

- In this directory, projects involving terraform will be dsplayed here along with a study guide on the Terraform Associate exam.

### What is Terraform?

- An open source IaC software tool that allows us to provision, manage, and change infrastructure through code, specifically HCL (Hashicorp Congiuration Language).

### Terraform Workflow

- 3 phases; write - plan - apply
- write - involves the starting off with version control software ie. github/gitlab
- plan - plane, review changes code will make to environment
- Apply - deploye changes to infractructure 


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