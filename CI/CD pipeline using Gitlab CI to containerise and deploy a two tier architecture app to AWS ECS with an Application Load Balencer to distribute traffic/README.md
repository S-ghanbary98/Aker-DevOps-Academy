# CI/CD pipeline using Gitlab CI to containerise and deploy a two tier architecture app to AWS ECS with an Application Load Balencer to distribute traffic.

## Intro

 In this project i'm going to build a CICD pipeline in aws using the following services

 - JavaScript
 - MongDb
 - Gitlab CI
 - AWS ECS
 - AWS VPC
 - AWS Application Load Balencer
 - Docker

 such that when a change is pushed to gitlab, this triggers the gitlab-ci pipeline build an image and update the container service, and task definition on EKS.




