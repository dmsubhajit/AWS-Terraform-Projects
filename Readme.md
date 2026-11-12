# AWS Terraform Portfolio --- Beginner to Pro

A hands-on Terraform portfolio for an AWS-focused DevOps Engineer. This
repository contains **four progressively harder, real-world projects**.
Build each project yourself, validate it, test failure scenarios,
document it, and tear it down safely.

> **Scope decision:** The enterprise multi-account / AWS Organizations
> project has been removed because it requires access to multiple AWS
> accounts and organization-level permissions. The remaining projects
> can be built in a personal AWS account, with some optional features
> requiring additional permissions or budget.

## Portfolio roadmap

  ------------------------------------------------------------------------------
  \#                Project           Level                    Main outcome
  ----------------- ----------------- ------------------------ -----------------
  1                 AWS               Beginner                 Secure VPC, EC2,
                    Infrastructure                             S3, IAM and
                    Foundation                                 remote Terraform
                                                               state

  2                 Highly Available  Beginner--Intermediate   Multi-AZ
                    Web Application                            application
                                                               infrastructure
                                                               with ALB, Auto
                                                               Scaling and RDS

  3                 Containerized     Intermediate             ECS Fargate, ECR,
                    Application                                microservices and
                    Platform                                   CI/CD

  4                 Production        Advanced / Pro           EKS, GitOps,
                    Kubernetes                                 observability,
                    Platform on AWS                            security and
                                                               disaster recovery
  ------------------------------------------------------------------------------

## How to use this repository

-   Keep all four projects in this **one GitHub repository**.
-   Complete tasks in order. Do not skip directly to copying a finished
    implementation.
-   Work locally, run the checks, deploy to AWS, test the result,
    document evidence, and clean up.
-   Commit small, meaningful changes. Never commit credentials,
    Terraform state, plan files containing secrets, or `.tfvars` files
    containing secrets.
-   Use separate state keys for each project and environment. Do not
    share one state file across unrelated projects.
-   Use Terraform modules when they improve reuse; do not abstract
    everything prematurely.

## Suggested repository structure

``` text
aws-terraform-portfolio/
├── README.md
├── .gitignore
├── docs/
│   ├── portfolio-roadmap.md
│   ├── cost-and-cleanup.md
│   └── decisions/
├── project-01-foundation/
│   ├── bootstrap/
│   ├── modules/
│   ├── environments/
│   │   └── dev/
│   ├── tests/
│   ├── docs/
│   └── README.md
├── project-02-high-availability/
│   ├── modules/
│   ├── environments/
│   │   └── dev/
│   ├── tests/
│   ├── docs/
│   └── README.md
├── project-03-ecs-platform/
│   ├── modules/
│   ├── environments/
│   │   ├── dev/
│   │   └── staging/
│   ├── app/
│   ├── .github/
│   │   └── workflows/
│   ├── tests/
│   ├── docs/
│   └── README.md
└── project-04-eks-platform/
    ├── modules/
    ├── environments/
    │   └── dev/
    ├── platform/
    ├── applications/
    ├── gitops/
    ├── .github/
    │   └── workflows/
    ├── tests/
    ├── docs/
    └── README.md
```

Adapt the structure as the project grows. Each project should have its
own README, architecture diagram, deployment instructions, validation
evidence, troubleshooting notes, and cleanup instructions.

------------------------------------------------------------------------

# Project 1 --- AWS Infrastructure Foundation

**Level:** Beginner\
**Scenario:** A small company needs a secure AWS network and a basic
compute/storage foundation provisioned entirely through Terraform.

## Target architecture

-   One custom VPC: `10.0.0.0/16`
-   Two Availability Zones
-   Two public subnets and two private subnets
-   Internet Gateway and route tables
-   NAT Gateway for private outbound internet access
-   Private EC2 instance managed through Systems Manager Session Manager
-   IAM role and instance profile
-   Encrypted, versioned S3 bucket
-   Basic CloudWatch monitoring
-   S3-backed Terraform remote state with locking

## Tasks

-   [ ] **1. Prepare your workstation**
    -   [ ] Install Terraform CLI, AWS CLI and Git.
    -   [ ] Configure AWS authentication using IAM Identity Center or
        another short-lived credential method.
    -   [ ] Verify identity with `aws sts get-caller-identity`.
    -   [ ] Create this GitHub repository and the project folder.
-   [ ] **2. Bootstrap Terraform state**
    -   [ ] Create a dedicated S3 state bucket.
    -   [ ] Enable bucket versioning and server-side encryption.
    -   [ ] Block public access.
    -   [ ] Enable S3 lockfile-based state locking
        (`use_lockfile = true`).
    -   [ ] Configure the backend only after the bucket exists.
-   [ ] **3. Build the network**
    -   [ ] Create the VPC and enable DNS support/hostnames.
    -   [ ] Create two public and two private subnets across two AZs.
    -   [ ] Create and attach an Internet Gateway.
    -   [ ] Configure public route tables.
    -   [ ] Create a NAT Gateway and private route tables.
    -   [ ] Associate each subnet with the correct route table.
-   [ ] **4. Deploy compute**
    -   [ ] Create an EC2 instance in a private subnet.
    -   [ ] Create a least-privilege IAM role and instance profile.
    -   [ ] Enable Systems Manager access without opening SSH to the
        internet.
    -   [ ] Configure security groups with minimum required access.
-   [ ] **5. Add storage and monitoring**
    -   [ ] Create an encrypted S3 application bucket with versioning.
    -   [ ] Block public access and configure lifecycle rules.
    -   [ ] Configure CloudWatch logs/metrics and at least one alarm.
-   [ ] **6. Improve Terraform quality**
    -   [ ] Use variables, outputs, locals and data sources.
    -   [ ] Use `for_each` for repeated subnets or related resources
        where appropriate.
    -   [ ] Add input validation and sensible defaults.
    -   [ ] Add common tags.
    -   [ ] Pin Terraform and provider version constraints.
    -   [ ] Extract reusable VPC and security modules.
    -   [ ] Add `dev.tfvars` without secrets.
-   [ ] **7. Test and document**
    -   [ ] Run `terraform fmt -recursive`.
    -   [ ] Run `terraform init`, `terraform validate` and
        `terraform plan`.
    -   [ ] Apply and verify resources in AWS.
    -   [ ] Test private EC2 access through Session Manager.
    -   [ ] Test S3 encryption, versioning and access restrictions.
    -   [ ] Destroy the environment and verify cleanup.

## Terraform concepts to demonstrate

Variables, outputs, locals, data sources, `for_each`, `count`, dynamic
blocks, conditional expressions, dependencies, lifecycle rules, state,
import, backend configuration, module inputs/outputs, and resource
tagging.

## Definition of done

-   [ ] Entire environment can be created from code.
-   [ ] No public SSH access is required.
-   [ ] Private instance can be reached through Systems Manager.
-   [ ] S3 bucket security controls are verified.
-   [ ] Remote state and locking work.
-   [ ] README, diagram, deployment steps and cleanup steps are
    committed.

------------------------------------------------------------------------

# Project 2 --- Highly Available Web Application

**Level:** Beginner--Intermediate\
**Scenario:** A customer-facing web application must tolerate an EC2
instance failure and be designed across multiple Availability Zones.

## Target architecture

-   VPC across two AZs
-   Public subnets for load balancers
-   Private application subnets
-   Isolated/private database subnets
-   Application Load Balancer (ALB)
-   EC2 Launch Template and Auto Scaling Group
-   RDS PostgreSQL with Multi-AZ deployment
-   Route 53 and ACM HTTPS
-   Optional CloudFront and S3 for static assets
-   Secrets Manager, CloudWatch and SNS notifications

## Tasks

-   [ ] **1. Network design**
    -   [ ] Create public, application-private and database subnets
        across two AZs.
    -   [ ] Configure routing and NAT for private outbound access.
    -   [ ] Ensure the database is not publicly accessible.
    -   [ ] Restrict traffic between tiers using security groups.
-   [ ] **2. Load balancing and compute**
    -   [ ] Create an Application Load Balancer.
    -   [ ] Create target groups and health checks.
    -   [ ] Create an EC2 Launch Template.
    -   [ ] Deploy an Auto Scaling Group across two AZs.
    -   [ ] Bootstrap the application using user data or a
        configuration-management approach.
    -   [ ] Configure scaling policies based on a measurable metric such
        as CPU.
-   [ ] **3. Database**
    -   [ ] Create an RDS subnet group.
    -   [ ] Deploy PostgreSQL with Multi-AZ enabled.
    -   [ ] Configure encryption and automated backups.
    -   [ ] Store credentials in Secrets Manager.
    -   [ ] Allow database access only from the application security
        group.
    -   [ ] Document backup retention and recovery assumptions.
-   [ ] **4. DNS and HTTPS**
    -   [ ] Request or import an ACM certificate for a domain you
        control.
    -   [ ] Configure an HTTPS ALB listener.
    -   [ ] Redirect HTTP to HTTPS.
    -   [ ] Configure Route 53 records.
    -   [ ] Optionally put CloudFront in front of the application and
        use S3 for static assets.
-   [ ] **5. Monitoring and resilience**
    -   [ ] Create CloudWatch dashboard and alarms.
    -   [ ] Configure SNS notifications.
    -   [ ] Enable ALB access logs.
    -   [ ] Test unhealthy-instance replacement.
    -   [ ] Test scaling out and scaling in.
    -   [ ] Perform a database backup/restore exercise.
    -   [ ] Document what an AZ failure would mean for each component.
-   [ ] **6. Advanced improvements**
    -   [ ] Add AWS WAF if budget and permissions allow.
    -   [ ] Add S3 lifecycle policies.
    -   [ ] Implement rolling or instance-refresh deployments.
    -   [ ] Add Terraform tests and security scanning.
    -   [ ] Document expected RTO and RPO, and validate what you can.

## Definition of done

-   [ ] Application is reachable over HTTPS.
-   [ ] ALB distributes traffic to healthy targets.
-   [ ] Failed instances are replaced automatically.
-   [ ] Database is private and encrypted.
-   [ ] Backup restoration is demonstrated.
-   [ ] Monitoring and alerting are verified.
-   [ ] Architecture, limitations, cost and recovery steps are
    documented.

------------------------------------------------------------------------

# Project 3 --- Containerized Application Platform

**Level:** Intermediate\
**Scenario:** A software team wants to deploy containerized services on
AWS without managing Kubernetes control planes or EC2 container hosts.

## Target architecture

-   Amazon ECR
-   ECS cluster using AWS Fargate
-   ECS services and task definitions
-   ALB and private subnets
-   RDS PostgreSQL
-   ElastiCache Redis (optional, cost-aware)
-   S3
-   Secrets Manager and KMS
-   CloudWatch Logs and alarms
-   GitHub Actions or GitLab CI with OIDC authentication

## Suggested application

Build or use a small application with these components:

-   Frontend
-   Backend API
-   Authentication service
-   PostgreSQL database
-   Optional Redis cache

Start with two services if building all components would slow down
infrastructure learning. Expand later.

## Tasks

-   [ ] **1. Container registry**
    -   [ ] Create an ECR repository for each service.
    -   [ ] Configure image scanning and lifecycle policies.
    -   [ ] Build and tag Docker images.
    -   [ ] Push images to ECR.
-   [ ] **2. ECS platform**
    -   [ ] Create an ECS cluster.
    -   [ ] Create task execution and task IAM roles.
    -   [ ] Create task definitions with CPU and memory settings.
    -   [ ] Deploy Fargate services in private subnets.
    -   [ ] Configure ALB listeners, target groups and health checks.
    -   [ ] Configure ECS service autoscaling.
    -   [ ] Configure container health checks and log groups.
-   [ ] **3. Application dependencies**
    -   [ ] Provision RDS PostgreSQL in private subnets.
    -   [ ] Store database credentials in Secrets Manager.
    -   [ ] Inject secrets into tasks securely.
    -   [ ] Optionally provision ElastiCache Redis.
    -   [ ] Configure S3 access through a task role.
-   [ ] **4. CI/CD**
    -   [ ] Create a pipeline triggered by a code push.
    -   [ ] Run tests and build the Docker image.
    -   [ ] Scan the image for vulnerabilities.
    -   [ ] Push the image to ECR.
    -   [ ] Register a new ECS task definition revision.
    -   [ ] Deploy the new revision to ECS.
    -   [ ] Verify service stability and health.
    -   [ ] Configure rollback or deployment circuit-breaker behavior.
-   [ ] **5. Secure the pipeline**
    -   [ ] Configure GitHub Actions/GitLab OIDC federation with AWS.
    -   [ ] Avoid long-lived AWS access keys in CI variables.
    -   [ ] Restrict the CI role to required ECR and ECS actions.
    -   [ ] Separate deployment permissions by environment.
-   [ ] **6. Advanced deployment**
    -   [ ] Configure blue/green deployment using CodeDeploy where
        appropriate.
    -   [ ] Add staging and production environment configuration.
    -   [ ] Add approval before production deployment.
    -   [ ] Add CloudWatch alarms and SNS notifications.
    -   [ ] Test failed deployment and rollback.

## Definition of done

-   [ ] Application runs on ECS Fargate.
-   [ ] Images are stored in ECR and scanned.
-   [ ] Tasks run in private subnets.
-   [ ] Secrets are not embedded in images or source code.
-   [ ] Code push triggers a working deployment pipeline.
-   [ ] Failed deployment behavior is tested.
-   [ ] Logs, alarms, deployment steps and cleanup are documented.

------------------------------------------------------------------------

# Project 4 --- Production Kubernetes Platform on AWS (EKS)

**Level:** Advanced / Pro\
**Scenario:** A company needs a production-style Kubernetes platform
with automated infrastructure provisioning, GitOps delivery,
observability, security controls and a tested recovery plan.

> This is the capstone. EKS, NAT Gateways, load balancers, managed
> databases and observability components can be expensive. Deploy a
> small development version first and destroy it when finished.

## Target architecture

-   VPC spanning multiple AZs
-   EKS cluster
-   Managed node groups; optionally Karpenter
-   ECR
-   AWS Load Balancer Controller
-   Route 53 and TLS
-   EBS CSI driver; optional EFS CSI driver
-   RDS PostgreSQL and optional ElastiCache
-   Secrets Manager with External Secrets
-   Argo CD and Helm
-   Prometheus and Grafana
-   Fluent Bit / CloudWatch logging
-   AWS Backup and cross-region recovery components where justified

## Tasks

-   [ ] **1. EKS infrastructure**
    -   [ ] Provision VPC and private subnets across multiple AZs.
    -   [ ] Provision EKS with Terraform.
    -   [ ] Restrict cluster API access appropriately.
    -   [ ] Create managed node groups.
    -   [ ] Configure EKS access entries and least-privilege access.
    -   [ ] Configure EKS Pod Identity or IRSA.
    -   [ ] Install and configure required cluster add-ons.
-   [ ] **2. Cluster networking and storage**
    -   [ ] Install AWS Load Balancer Controller.
    -   [ ] Configure ingress and ALB routing.
    -   [ ] Configure Route 53 integration.
    -   [ ] Configure TLS certificates.
    -   [ ] Install EBS CSI driver.
    -   [ ] Test persistent volume provisioning and recovery.
    -   [ ] Add EFS only if the application needs shared file storage.
-   [ ] **3. Workload deployment**
    -   [ ] Deploy application services using Helm.
    -   [ ] Configure namespaces and resource quotas.
    -   [ ] Configure ConfigMaps and externalized secrets.
    -   [ ] Set CPU/memory requests and limits.
    -   [ ] Configure startup, readiness and liveness probes.
    -   [ ] Configure Horizontal Pod Autoscaler.
    -   [ ] Configure Pod Disruption Budgets.
    -   [ ] Apply network policies where supported by the chosen
        networking setup.
-   [ ] **4. GitOps and delivery**
    -   [ ] Store Kubernetes manifests or Helm values in Git.
    -   [ ] Install Argo CD.
    -   [ ] Configure GitOps synchronization.
    -   [ ] Separate development and production configuration.
    -   [ ] Configure promotion through pull requests.
    -   [ ] Implement rolling deployment.
    -   [ ] Demonstrate canary or blue/green delivery.
    -   [ ] Test rollback to a known-good release.
-   [ ] **5. Autoscaling**
    -   [ ] Configure HPA for application pods.
    -   [ ] Configure node autoscaling.
    -   [ ] Optionally implement Karpenter.
    -   [ ] Load-test the application.
    -   [ ] Verify scale-out and scale-in behavior.
    -   [ ] Set sensible limits to control unexpected cost.
-   [ ] **6. Observability**
    -   [ ] Deploy Prometheus.
    -   [ ] Deploy Grafana dashboards.
    -   [ ] Collect application and cluster logs.
    -   [ ] Configure alerts for workload and node health.
    -   [ ] Monitor API latency, error rate and resource saturation.
    -   [ ] Document useful operational queries and dashboards.
-   [ ] **7. Security**
    -   [ ] Use IAM roles for workloads instead of static AWS
        credentials.
    -   [ ] Store secrets in Secrets Manager.
    -   [ ] Apply least-privilege Kubernetes RBAC.
    -   [ ] Scan container images.
    -   [ ] Scan Terraform and Kubernetes configuration.
    -   [ ] Encrypt sensitive data and restrict network paths.
    -   [ ] Review audit logs and cluster access.
-   [ ] **8. Disaster recovery**
    -   [ ] Define RTO and RPO for the application.
    -   [ ] Configure database backups.
    -   [ ] Test database restoration.
    -   [ ] Configure S3 replication if object recovery requires it.
    -   [ ] Design a secondary-region recovery approach.
    -   [ ] Document failover and failback steps.
    -   [ ] Run a controlled recovery exercise.
-   [ ] **9. Failure testing**
    -   [ ] Delete a test pod and verify replacement.
    -   [ ] Drain a node and verify workload rescheduling.
    -   [ ] Test failed deployment and rollback.
    -   [ ] Test database restore.
    -   [ ] Test recovery from a Terraform-managed resource change.
    -   [ ] Record expected versus observed recovery behavior.

## Definition of done

-   [ ] EKS cluster is provisioned through Terraform.
-   [ ] Workloads deploy through GitOps.
-   [ ] Autoscaling works and is demonstrated.
-   [ ] Logs, metrics and alerts are usable.
-   [ ] Workload identity and access controls are configured.
-   [ ] Backup restoration and a recovery exercise are documented.
-   [ ] Teardown and cost controls are verified.

------------------------------------------------------------------------

# Standards required across every project

Use these standards progressively. Project 1 should demonstrate the
basics; later projects should implement the full workflow.

  -----------------------------------------------------------------------
  Area                                Expected practice
  ----------------------------------- -----------------------------------
  Terraform version                   Pin Terraform version constraints

  Providers                           Pin provider version constraints
                                      and commit `.terraform.lock.hcl`

  Formatting                          `terraform fmt -recursive`

  Validation                          `terraform validate`

  Linting                             TFLint

  Security                            Checkov or Trivy configuration
                                      scanning

  Testing                             Terraform tests; Terratest where
                                      useful

  State                               S3 remote state, versioning,
                                      encryption and lockfile locking

  Identity                            Short-lived credentials; OIDC for
                                      CI/CD

  Secrets                             Secrets Manager or another managed
                                      secret store

  Modules                             Small, reusable, documented modules

  CI/CD                               Plan on pull request; controlled
                                      apply after approval

  Drift                               Scheduled plan/drift checks

  Cost                                AWS Budgets and cost estimation

  Documentation                       README, architecture diagram,
                                      decisions and runbooks

  Operations                          Logs, metrics, alarms, backup and
                                      recovery tests
  -----------------------------------------------------------------------

## Example Terraform validation workflow

Run these locally from the relevant project directory:

``` bash
terraform fmt -recursive
terraform init
terraform validate
terraform plan
```

Add linting and security checks as you introduce them:

``` bash
tflint --init
tflint

checkov -d .
```

Do not run `terraform apply` automatically on every branch. Use
pull-request plans and a controlled approval process for deployments.

## State management notes

For new Terraform S3 backends:

-   Enable S3 bucket versioning.
-   Enable encryption and block public access.
-   Use a separate state key per project/environment.
-   Use `use_lockfile = true` for S3-native state locking.
-   Restrict state access because state may contain sensitive values.
-   Do not commit state files or plan files to Git.

Example backend configuration (replace the bucket and key with your own
values):

``` hcl
terraform {
  backend "s3" {
    bucket       = "REPLACE_WITH_YOUR_STATE_BUCKET"
    key          = "project-01/dev/terraform.tfstate"
    region       = "ap-south-1"
    encrypt      = true
    use_lockfile = true
  }
}
```

The backend bucket must exist before `terraform init` can initialize
this backend. Keep bootstrap infrastructure separate from the state it
creates.

## Cost and safety checklist

Before every deployment:

-   [ ] Confirm the AWS account and region.
-   [ ] Check the Terraform plan for unexpected replacements or
    deletions.
-   [ ] Set AWS Budget alerts.
-   [ ] Review NAT Gateway, RDS, EKS, load balancer, logging and
    data-transfer costs.
-   [ ] Use small development resources where possible.
-   [ ] Avoid leaving temporary environments running.
-   [ ] Back up data before destructive changes.

After every project session:

-   [ ] Record resources intentionally left running.
-   [ ] Destroy temporary infrastructure when it is no longer needed.
-   [ ] Verify there are no orphaned resources.
-   [ ] Keep required state/bootstrap resources until dependent state is
    safely retired.
-   [ ] Record any unexpected charges or cleanup issues.

Never use `terraform destroy` blindly against shared or production
resources. Review the plan and confirm the target workspace, account,
region and state first.

------------------------------------------------------------------------

# Portfolio progress tracker

Update this section as you complete each project.

  Project                           Status        Completion date   Evidence / PR
  --------------------------------- ------------- ----------------- ---------------
  Project 1 --- Foundation          Not started   ---               ---
  Project 2 --- High Availability   Not started   ---               ---
  Project 3 --- ECS Platform        Not started   ---               ---
  Project 4 --- EKS Platform        Not started   ---               ---

## Project review template

Copy this section into each project's README when you finish:

``` markdown
## Project review

### Architecture
- Architecture diagram:
- AWS region:
- Main services:

### Implementation
- Terraform modules:
- Remote state:
- CI/CD:
- Security controls:

### Validation evidence
- Terraform fmt:
- Terraform validate:
- Terraform plan:
- Security scan:
- Tests:

### Failure and recovery tests
- Failure simulated:
- Expected behavior:
- Observed behavior:
- Recovery steps:
- RTO/RPO result, if applicable:

### Cost and cleanup
- Approximate cost:
- Resources destroyed:
- Persistent resources retained:
- Cleanup verification:

### Lessons learned
- What worked:
- What failed:
- What I would improve:
```

## Final portfolio goal

By completing these four projects, you should have evidence that you
can:

1.  Design and provision AWS infrastructure using Terraform.
2.  Build secure, multi-AZ application infrastructure.
3.  Automate container builds and deployments on ECS.
4.  Build and operate a Kubernetes platform on EKS.
5.  Implement remote state, modules, testing, security scanning and
    CI/CD.
6.  Troubleshoot failures, restore services, control costs and document
    operations.

**Working agreement:** Build the infrastructure on your own machine.
Work through one task at a time. Keep the code and evidence in this
single GitHub repository. Request a code review when you finish each
task, and move to the next task only after the current one meets its
completion criteria.
