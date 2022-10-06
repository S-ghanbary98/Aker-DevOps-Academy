# CI/CD CodePipeline Project

## Intro

 In this project i'm going to build a CICD pipeline in aws using the following aws services

 - CodePipeline
 - CodeaDeploy
 - CodeCommit
 - EC2
 - IAM

 such that when a change is pushed to code commit this triggers the pipeline to push code to the ec2 instance.


 ## CodeCommit 

 - Firstly we have to build a CodeCommit depository. By entering CodeCommit, and clicking on 'create reporsitory', afterwhich we give a name to our repository and finally click on create.

![repository](img/codecommit.png)
![repository](img/repo.png)

- Next in our console we have to go to IAM and create a user. This is the user we will use to make changes so once created we will have to grant it the 'codecommit power use' permissions policy, so it may access codecommit and make commits.


![repository](img/iam_user.png)

- Next we sign in as our IAM user, and back in IAM we create https credentials for access to code commit.

![http credentials](img/codecommit_creds.png)

- We can now clone our repository, add your code files, add to the staging area, and finally commit and make a push to code commit via `git push origin master`

![commit](img/commit.png)