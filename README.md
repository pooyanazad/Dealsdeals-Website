# DealsDeals Infrastructure and Deployment

This project contains infrastructure as code (Terraform) and deployment configurations for the DealsDeals application.


## Infrastructure Overview

The infrastructure is split into two environments:
- Test Environment: Basic EKS setup with minimal redundancy
- Production Environment: Highly available EKS setup with multi-AZ deployment

### Key Features

#### Test Environment
- Single NAT Gateway
- 2 Availability Zones
- t3.small nodes
- 1-3 nodes autoscaling

#### Production Environment
- Multiple NAT Gateways
- 3 Availability Zones
- t3.medium/t3.large nodes
- 2-5 nodes autoscaling
- VPC Flow Logs enabled
- Stricter security configurations

## Prerequisites

- AWS CLI configured
- Terraform >= 1.3.0
- kubectl (You should have it to connect, run and test infra on your local machine, Its possible to insatll with snap easily)
- Docker

## Initial Setup

1. Create S3 bucket for Terraform state: (You should do it to be safe but upload is denied in gitignore file, if just want to TEST it)
```bash
aws s3api create-bucket \
    --bucket dealsdeals-terraform-state \
    --region us-east-1

aws s3api put-bucket-versioning \
    --bucket dealsdeals-terraform-state \
    --versioning-configuration Status=Enabled
```

2. Create DynamoDB table for state locking:
```bash
aws dynamodb create-table \
    --table-name terraform-state-lock \
    --attribute-definitions AttributeName=LockID,AttributeType=S \
    --key-schema AttributeName=LockID,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
```

## Deployment Process

1. First-time infrastructure setup:
```bash
# Test environment
cd infra/test
terraform init
terraform plan
terraform apply

# Production environment
cd infra/prod
terraform init
terraform plan
terraform apply
```

2. Create Kubernetes namespaces:
```bash
kubectl create namespace test
kubectl create namespace prod
```

3. Deploy application:
```bash
# Test deployment
kubectl apply -f k8s/test/deployment.yaml
kubectl apply -f k8s/test/service.yaml

# Production deployment
kubectl apply -f k8s/prod/deployment.yaml
kubectl apply -f k8s/prod/service.yaml
```

## Continuous Deployment

The project uses GitHub Actions for CI/CD. The pipeline:
1. Runs tests
2. Builds and tags Docker image
3. Deploys to test environment
4. Requires manual approval for production
5. Deploys to production

## Monitoring and Maintenance

To check deployment status:
```bash
# Test environment
kubectl get all -n test

# Production environment
kubectl get all -n prod
```

## Security Notes

1. Production environment includes:
   - Restricted API access CIDRs
   - VPC Flow Logs
   - Multiple NAT Gateways
   - Larger instance types
   - Higher redundancy

2. Security groups are configured for:
   - HTTP (80)
   - HTTPS (443)
   - All outbound traffic

## Version Management

- Application versions are tracked in VERSION file
- Each deployment creates both versioned and latest tags
- Production deployments require explicit approval
