# EC2 + S3 + CloudWatch Monitoring Stack

CloudFormation template that deploys a reference architecture on AWS,
applying security and observability best practices.

## What it includes

- **EC2**: instance with AMI automatically resolved via SSM Parameter Store
  (always the latest available Amazon Linux 2023, no hardcoded AMI IDs).
- **Least-privilege IAM Role**: the instance can only read/write to its
  associated S3 bucket and publish metrics/logs — nothing else.
- **S3**: bucket with versioning enabled, default encryption (SSE-S3), and
  full public access block.
- **CloudWatch Alarms**: alerts for high CPU (>80% sustained) and failed
  status checks, notifying via email through SNS.
- **Security Group**: SSH access restricted to a single port (meant to be
  scoped to a specific CIDR in production).

## Usage

```bash
aws cloudformation deploy \
  --template-file ec2-s3-monitoring-stack.yaml \
  --stack-name dev-cloudops-stack \
  --parameter-overrides \
      EnvironmentName=dev \
      KeyPairName=your-keypair \
      AlarmNotificationEmail=your-email@example.com \
  --capabilities CAPABILITY_NAMED_IAM
```

## Parameters

| Parameter | Description | Default |
|---|---|---|
| `EnvironmentName` | Environment prefix (dev/staging/prod) | `dev` |
| `InstanceType` | EC2 instance type | `t3.micro` |
| `KeyPairName` | Existing key pair for SSH access | — |
| `AlarmNotificationEmail` | Email for CloudWatch alerts | — |

## Design rationale

This template reflects the approach I apply in production environments:
IAM roles scoped to the minimum required, encryption and public access
blocking enabled by default on S3, and active monitoring from day one —
rather than bolted on as an afterthought.