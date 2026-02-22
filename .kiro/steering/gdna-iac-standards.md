---
title: gdna-iac-standards
inclusion: always
---

# g/d/n/a Infrastructure as Code Standards

## IaC Tool Choice

Like the frontend framework choice (Vite for fast/simple, Next.js for robust/complex), the IaC tool is chosen during the AIDLC spec process based on engagement context. Both paths share the same tagging, security, testing, and deployment standards below.

### AWS CDK (TypeScript) — Default

Use when:
- AWS-only deployment
- g/d/n/a is greenfield or owns the infrastructure decisions
- Agentic architecture generation is part of the workflow
- Team is TypeScript-primary (shared language with frontend)

Why:
- L2/L3 constructs enforce security and compliance by default
- TypeScript CDK shares language competency with the rest of the stack
- Agentic CDK capabilities align with AI-driven architecture generation
- CDK Pipelines for self-mutating deployment

### Terraform — When Required

Use when:
- Customer has existing Terraform estate that must be maintained
- Multi-cloud deployment (Azure/GCP alongside AWS)
- Customer mandate or strong team preference for HCL
- Regulatory or procurement requirement for Terraform

When using Terraform:
- CDKTF (CDK for Terraform) as bridge when the team is TypeScript-native
- Standard HCL when the customer's team will maintain post-engagement
- Terraform Cloud or S3 backend for state — never local state files
- Module registry for reusable components (mirrors CDK construct library pattern)

### g/d/n/a Reusable Architecture Modules

Regardless of CDK or Terraform, g/d/n/a modules follow the same contract:
- Defined inputs (config), outputs (endpoints, ARNs), tagging, security baseline
- Natural language architecture generation targets these module interfaces
- Modules are versioned and published to internal registry
- Same security defaults enforced in both CDK constructs and TF modules

## Project Structure

### CDK Structure (packages/infra)
```
packages/infra/
├── bin/
│   └── app.ts                  # CDK app entry point
├── lib/
│   ├── stacks/                 # Stack definitions
│   │   ├── network-stack.ts
│   │   ├── compute-stack.ts
│   │   ├── data-stack.ts
│   │   └── monitoring-stack.ts
│   ├── constructs/             # Reusable L3 constructs
│   │   ├── secure-api.ts
│   │   ├── compliant-bucket.ts
│   │   └── audited-lambda.ts
│   └── config/                 # Environment configurations
│       ├── environments.ts
│       └── tags.ts
├── test/
│   ├── stacks/
│   └── constructs/
├── cdk.json
└── tsconfig.json
```

### Terraform Structure (packages/infra)
```
packages/infra/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
├── modules/                    # Reusable modules
│   ├── networking/
│   ├── compute/
│   ├── data/
│   └── monitoring/
├── shared/                     # Shared configurations
│   ├── tags.tf                 # Tagging defaults
│   ├── providers.tf
│   └── backend.tf
└── test/
    └── *.tftest.hcl            # Terraform native tests
```

## Stack Organization
- **One stack per deployment boundary** — resources that deploy together belong together
- **Cross-stack references via interfaces** — never import concrete stack classes (CDK) or hardcode outputs (TF)
- **Environment-aware configuration** — dev/staging/prod differences via config, not conditionals
- **Stack/workspace names include environment** — `gdna-{customer}-{workload}-{env}`

## Mandatory Tagging

Seven tags. Required on every stack. Synth/plan fails without them.

| Tag | What it answers | Examples |
|-----|----------------|----------|
| `gdna:deployed-by` | Who built this? | Always `gdna` |
| `gdna:customer` | End customer? | `rekalibrate`, `growthbook`, `internal` |
| `gdna:engagement` | Which engagement? | `MAP-abc123`, `ASA-REKAL-001` |
| `gdna:workload` | What system? | `qbr-engine`, `guardian`, `matchmaker` |
| `gdna:module` | Which piece? | `data-ingestion`, `auth`, `hubspot-sync` |
| `gdna:env` | Where running? | `prod`, `dev`, `staging`, `demo`, `buildlearn` |
| `gdna:grc` | Compliance scope? | `ftr`, `wafr`, `soc2`, `hipaa`, `none` |

### Tagging Rules
- All values lowercase, hyphenated
- Tag at the App/Stack level — everything inherits automatically
- Don't tag individual resources
- `internal` as customer for g/d/n/a's own R&D and templates
- `none` for grc when no compliance framework applies
- Activate all `gdna:*` tags in AWS Billing → Cost Allocation Tags or they won't appear in Cost Explorer

### CDK Tagging Implementation
```typescript
// infra/lib/config/tags.ts
export interface GdnaTagConfig {
  customer: string;
  engagement: string;
  workload: string;
  module: string;
  env: string;
  grc: string;
}

export function applyTags(scope: Construct, config: GdnaTagConfig) {
  const tags = cdk.Tags.of(scope);
  tags.add('gdna:deployed-by', 'gdna');
  tags.add('gdna:customer', config.customer);
  tags.add('gdna:engagement', config.engagement);
  tags.add('gdna:workload', config.workload);
  tags.add('gdna:module', config.module);
  tags.add('gdna:env', config.env);
  tags.add('gdna:grc', config.grc);
}

// bin/app.ts
const app = new cdk.App();
applyTags(app, {
  customer: app.node.tryGetContext('customer') || 'internal',
  engagement: app.node.tryGetContext('engagement') || 'unassigned',
  workload: app.node.tryGetContext('workload') || 'unnamed',
  module: app.node.tryGetContext('module') || 'core',
  env: app.node.tryGetContext('env') || 'dev',
  grc: app.node.tryGetContext('grc') || 'none',
});
```

### Terraform Tagging Implementation
```hcl
# shared/tags.tf
locals {
  required_tags = {
    "gdna:deployed-by" = "gdna"
    "gdna:customer"    = var.customer
    "gdna:engagement"  = var.engagement
    "gdna:workload"    = var.workload
    "gdna:module"      = var.module_name
    "gdna:env"         = var.env
    "gdna:grc"         = var.grc
  }
}

# Apply to all resources via default_tags
provider "aws" {
  region = var.region
  default_tags {
    tags = local.required_tags
  }
}
```

### Tag Enforcement

CDK — Aspect that fails synthesis:
```typescript
const REQUIRED = [
  'gdna:deployed-by', 'gdna:customer', 'gdna:engagement',
  'gdna:workload', 'gdna:module', 'gdna:env', 'gdna:grc',
];

export class TagEnforcement implements IAspect {
  visit(node: IConstruct): void {
    if (Stack.isStack(node)) {
      const tagKeys = node.tags.renderTags().map((t: any) => t.key);
      for (const required of REQUIRED) {
        if (!tagKeys.includes(required)) {
          Annotations.of(node).addError(`Missing required tag: ${required}`);
        }
      }
    }
  }
}
// cdk.Aspects.of(app).add(new TagEnforcement());
```

Terraform — validation in variables:
```hcl
variable "customer" {
  type = string
  validation {
    condition     = can(regex("^[a-z][a-z0-9-]*$", var.customer))
    error_message = "Customer must be lowercase, hyphenated."
  }
}
# Repeat for each required tag variable
```

## Construct Patterns (CDK)

Every construct should enforce security and compliance defaults:
```typescript
export class CompliantBucket extends Construct {
  public readonly bucket: s3.Bucket;

  constructor(scope: Construct, id: string, props: CompliantBucketProps) {
    super(scope, id);
    this.bucket = new s3.Bucket(this, 'Bucket', {
      encryption: s3.BucketEncryption.S3_MANAGED,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      enforceSSL: true,
      versioned: true,
      removalPolicy: props.retainOnDelete
        ? cdk.RemovalPolicy.RETAIN
        : cdk.RemovalPolicy.DESTROY,
      serverAccessLogsBucket: props.accessLogBucket,
      lifecycleRules: props.dataRetentionDays
        ? [{ expiration: cdk.Duration.days(props.dataRetentionDays) }]
        : undefined,
    });
  }
}
```

## Module Patterns (Terraform)

Mirror the CDK construct pattern — secure defaults baked in:
```hcl
# modules/storage/main.tf
resource "aws_s3_bucket" "this" {
  bucket = var.bucket_name
}

resource "aws_s3_bucket_public_access_block" "this" {
  bucket                  = aws_s3_bucket.this.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_server_side_encryption_configuration" "this" {
  bucket = aws_s3_bucket.this.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
    }
  }
}

resource "aws_s3_bucket_versioning" "this" {
  bucket = aws_s3_bucket.this.id
  versioning_configuration {
    status = "Enabled"
  }
}
```

## Security Defaults (Non-Negotiable, Both Tools)
- **S3:** Block all public access, enforce SSL, enable versioning
- **Lambda:** VPC-attached when accessing data stores, least-privilege IAM
- **API Gateway:** WAF attached, throttling configured, access logging enabled
- **RDS/Aurora:** Encryption at rest, no public accessibility, automated backups
- **Secrets:** AWS Secrets Manager — never SSM Parameter Store for secrets
- **KMS:** Customer-managed keys for data classified CONFIDENTIAL or above
- **CloudTrail:** Enabled with log file validation, multi-region
- **VPC:** No default VPC usage, private subnets for compute, NAT Gateway for outbound

## IAM Patterns
- **Least privilege always** — Start with no permissions, add explicitly
- **No inline policies** — Use managed policies attached to roles
- **No wildcard resources** — Scope to specific ARNs
- **Service-linked roles** where available
- **Permission boundaries** for developer/deployment roles
- **Conditions** for cross-account access and MFA enforcement

```typescript
// CDK — ✅ Good — scoped permissions
lambdaFunction.addToRolePolicy(new iam.PolicyStatement({
  actions: ['dynamodb:GetItem', 'dynamodb:PutItem'],
  resources: [table.tableArn],
}));

// CDK — ❌ Bad — overly broad
lambdaFunction.addToRolePolicy(new iam.PolicyStatement({
  actions: ['dynamodb:*'],
  resources: ['*'],
}));
```

```hcl
# Terraform — ✅ Good — scoped permissions
data "aws_iam_policy_document" "lambda" {
  statement {
    actions   = ["dynamodb:GetItem", "dynamodb:PutItem"]
    resources = [aws_dynamodb_table.this.arn]
  }
}

# Terraform — ❌ Bad — overly broad
data "aws_iam_policy_document" "lambda_bad" {
  statement {
    actions   = ["dynamodb:*"]
    resources = ["*"]
  }
}
```

## Testing

### CDK Testing
- Snapshot tests for every stack — catch unintended changes
- Fine-grained assertions for security-critical resources
- Validation tests for construct input constraints
- Run with: `pnpm turbo test --filter=infra`

```typescript
test('S3 bucket blocks public access', () => {
  const template = Template.fromStack(stack);
  template.hasResourceProperties('AWS::S3::Bucket', {
    PublicAccessBlockConfiguration: {
      BlockPublicAcls: true,
      BlockPublicPolicy: true,
      IgnorePublicAcls: true,
      RestrictPublicBuckets: true,
    },
  });
});
```

### Terraform Testing
- `terraform validate` on every commit
- `terraform plan` diff review in PRs
- Native test framework (`*.tftest.hcl`) for module validation
- `tflint` for linting, `tfsec`/`checkov` for security scanning

```hcl
# test/s3.tftest.hcl
run "bucket_is_encrypted" {
  command = plan
  assert {
    condition     = aws_s3_bucket_server_side_encryption_configuration.this.rule[0].apply_server_side_encryption_by_default[0].sse_algorithm == "aws:kms"
    error_message = "Bucket must use KMS encryption"
  }
}
```

## Synth/Plan Validation
- CDK: `pnpm turbo cdk:synth --filter=infra` must pass before any commit (see `monorepo-standards.md`)
- Terraform: `terraform validate && terraform plan` must pass before any commit
- CDK: Use `cdk-nag` for automated security and compliance checking
- Terraform: Use `tfsec` or `checkov` for equivalent scanning
- Suppress rules only with documented justification
- Aspects (CDK) / Sentinel policies (TF Cloud) for organization-wide enforcement

## Deployment Pipeline
- CDK: CDK Pipelines for self-mutating deployment
- Terraform: Terraform Cloud workspaces or GitHub Actions with plan/apply stages
- Environment promotion: dev → staging → prod
- Manual approval gate before production
- Rollback strategy defined per stack/workspace
- Drift detection enabled in production

## Client Segment Considerations

### Startups
- Single-stack deployments acceptable
- Serverless-first (Lambda, DynamoDB, S3)
- Cost optimization: spot instances, reserved capacity planning
- CDK preferred — faster iteration, less configuration overhead

### SMB Transformation
- Multi-stack with shared networking
- Managed services preferred over self-hosted
- Backup and disaster recovery configured
- CDK or Terraform based on customer's existing tooling

### Enterprise / Agentic
- Multi-account strategy (AWS Organizations)
- Service Control Policies for guardrails
- Transit Gateway for network connectivity
- Centralized logging and monitoring account
- Terraform more common here due to existing enterprise TF estates

### GRC / Compliance-Heavy
- AWS Config rules for continuous compliance
- GuardDuty and Security Hub enabled
- Macie for PII detection in S3
- Evidence collection automated for audit readiness
- Both tools: compliance-as-code via cdk-nag (CDK) or Sentinel/checkov (TF)
