**CI/CD Workflow Architecture**

This is the demonstration for a CI/CD workflow for deploying a Kubernetes-based microservices application on AWS infrastructure using GitLab CI.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Dependencies](#dependencies)
3. [Assumptions](#assumptions)
4. [Setup and Configuration](#setup-and-configuration)
5. [Solution Diagram](#solution-diagram)
6. [Pipeline Stages](#pipeline-stages)
7. [Kubernetes Manifests](#kubernetes-manifests)
8. [Conclusion](#conclusion)

## Architecture Overview
Source Code is stored in a version control system like Git. GitLab is used for continuous integration and continuous deployment. 
We use AWS CDK for defining cloud infrastructure and AWS ECR for storing docker images. AWS EKS is used for running Kubernetes Cluster.
Kubernetes manifests are used to deploy the microservices.

## Dependencies
To make sure the microservice is deployed successfully with necessary access permissions, the following has to be setup.

- AWS account has to be created with IAM permissions for CDK and ECR.
- Kubernetes cluster (Amazon EKS).
- Necessary code pushed to the microservice application in Git account.
- SonarQube, Prisma for Quality Gates and security vulnerabilities
- Datadog, Splunk for monitoring

## Assumptions
- Gitlab account setup with proper permissions.
- AWS credentials and necessary permissions are configured and stored in GitLab CI/CD variable section.
- Kubernetes cluster set up on AWS EKS for all environments

## Setup and Configuration
Make sure to set up AWS account, Git account, ECR, EKS, secrets, AWS CLI, kubectl, Kustomize installed.

## Solution Diagram
Refer to solution diagram [here](SolutionDiagram.jpg)

## Pipeline Stages
Gitlab pipeline script - [gitlab-ci.yml](.gitlab-ci.yml)

- Developers push their code to central repository(GitLab/GitHub)
- Checkout and compiles code using GitLab and build artifacts(JAR files, docker images)
- SonarQube analyze code for potential bugs, code smells
- Enforce quality thresholds to ensure code meets required standards
- Prisma scan to ensure no image vulnerabilities
- Run all kinds of tests like Unit Tests, Integration Tests, E2E Test, Performance Tests
- Versioning - cut the release version, and Store it in ECR.
- Deploy the app to QA, Staging environment and followed by production environment
- Deployment is done by updating kubeconfig to EKS cluster and retrieves docker image from ECR
- Then it updates Kustomize config with the new tag and applies that config to deploy the application to the specified environment
- Monitor application using tools like splunk, Datadog and AWS cloud watch

**Note:** This implementation only have kustomize/overlay configuration for prod deploy as of now. Similarly, we can create the configuration for all other environments.

## Kubernetes Manifests

**Role Management**

- Create Role and RoleBinding to set permissions within a specific namespace
- Create ClusterRole and ClusterRoleBinding to set permissions across cluster

Refer to [rbac.yaml](rbac.yaml)

- Implement AWS Security groups to control inbound and outbound traffic to EKS nodes and pods
- Implement network policies to restrict pod to pod communication
- Ensure minimum permissions are given to users/groups
- Enable kubernetes audit logs and monitor activities of users and apps
- Use kubernetes secrets and AWS secret manager for sensitive data

**Auto Scaling**

- Create HPA using yaml -> [hpa.yaml](hpa.yaml)
- Enable cluster auto scaler to scale worker nodes based on pod demands and cluster resource utilization
- Configure Application Load balancer in front of the service to distribute traffic across healthy pods managed by HPA
- Monitor and refine scaling policies for optimal resource allocation


**Seamless deploy and availability**

Refer to [deployment.yaml](deployment.yaml) and [pdb.yaml](pdb.yaml)

- **Fault Tolerance at Cluster level** - Deploying kubernetes cluster across multiple AZ’s or regions
- **Load Balancing** to distribute traffic across cluster
- Implement **liveness and readiness** to automatically restart pods that are unhealthy and ensures traffic diverts to healthy pods

## Conclusion

This provides an overview of the CI/CD workflow architecture for a Kubernetes based microservices application using AWS infrastructure.
