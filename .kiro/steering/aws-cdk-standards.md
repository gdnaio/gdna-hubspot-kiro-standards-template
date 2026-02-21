---
title: AWS CDK & Infrastructure Standards
inclusion: always
---

# g/d/n/a AWS CDK & Infrastructure Standards

## Infrastructure Tool Decision

### Default: AWS CDK (TypeScript)
- All AWS-native client projects use CDK
- Agentic CDK capabilities align with AI-driven architecture generation
- TypeScript CDK shares language with frontend — single team competency
- L2/L3 constructs preferred over L1 (CloudFormation primitives)

### When Terraform is Required
- Multi-cloud deployments (client has Azure/GCP alongside AWS)
- Client mandate / existing TF estate that must be maintained
- When using Terraform: use CDKTF (CDK for Terraform) as bridge when possible

### g/d/n/a Reusable Architecture Modules
- Reusable modules provide a standardized interface regardless of backend (CDK or TF)
- Module contract: defined inputs (config), outputs (endpoints, ARNs), tagging, security baseline
- Natural language architecture generation targets these module interfaces
- Modules are versioned and published to internal registry

## CDK Project Structure
```
infra/
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
├── test/                       # CDK tests
│   ├── stacks/
│   └── constructs/
├── cdk.json
└── tsconfig.json
```

## Stack Organization
- **One stack per deployment boundary** — resources that deploy together belong together
- **Cross-stack references via interfaces** — never import concrete stack classes
- **Environment-aware configuration** — dev/staging/prod differences via config, not conditionals
- **Stack names include environment** — `gdna-{client}-{service}-{env}`

## Construct Patterns

### Compliant S3 Bucket (Example L3 Construct)
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

## Mandatory Tagging
Every resource must be tagged. Applied at the App level:
```typescript
const app = new cdk.App();
cdk.Tags.of(app).add('gdna:project', projectName);
cdk.Tags.of(app).add('gdna:environment', environment);
cdk.Tags.of(app).add('gdna:owner', teamOwner);
cdk.Tags.of(app).add('gdna:cost-center', costCenter);
cdk.Tags.of(app).add('gdna:data-classification', dataClassification);
cdk.Tags.of(app).add('gdna:compliance', complianceFramework);
```

## Security Defaults (Non-Negotiable)
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
// ✅ Good — scoped permissions
lambdaFunction.addToRolePolicy(new iam.PolicyStatement({
  actions: ['dynamodb:GetItem', 'dynamodb:PutItem'],
  resources: [table.tableArn],
}));

// ❌ Bad — overly broad
lambdaFunction.addToRolePolicy(new iam.PolicyStatement({
  actions: ['dynamodb:*'],
  resources: ['*'],
}));
```

## CDK Testing
- **Snapshot tests** for every stack — catch unintended changes
- **Fine-grained assertions** for security-critical resources
- **Validation tests** for construct input constraints
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

## CDK Synth Validation
- `pnpm turbo cdk:synth --filter=infra` must pass before any commit (see `monorepo-standards.md` for Turbo pipeline)
- Use `cdk-nag` for automated security and compliance checking
- Suppress nag rules only with documented justification
- Aspects for organization-wide policy enforcement

## Deployment Pipeline
- CDK Pipelines for self-mutating deployment
- Environment promotion: dev → staging → prod
- Manual approval gate before production
- Rollback strategy defined per stack
- Drift detection enabled in production

## Client Segment Considerations

### Startups
- Single-stack deployments acceptable
- Serverless-first (Lambda, DynamoDB, S3)
- Cost optimization: spot instances, reserved capacity planning

### SMB Transformation
- Multi-stack with shared networking
- Managed services preferred over self-hosted
- Backup and disaster recovery configured

### Enterprise / Agentic
- Multi-account strategy (AWS Organizations)
- Service Control Policies for guardrails
- Transit Gateway for network connectivity
- Centralized logging and monitoring account

### GRC / Compliance-Heavy
- AWS Config rules for continuous compliance
- GuardDuty and Security Hub enabled
- Macie for PII detection in S3
- Evidence collection automated for audit readiness
