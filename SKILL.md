---
name: poly-scanner
description: AI-powered infrastructure scanner for automated detection, analysis, and reporting of security, compliance, and cost issues across cloud and on-prem environments
version: 2.4.1
author: OpenClaw Infrastructure Team
tags:
  - infrastructure
  - ai
  - automation
  - security
  - compliance
  - cost-optimization
dependencies:
  - python>=3.9
  - boto3>=1.34.0
  - kubernetes>=28.1.0
  - docker>=6.1.3
  - ansible-core>=2.15.0
  - tfsec>=0.1.58
  - checkov>=3.2.0
  - pylint>=3.0.0
  - openai>=1.0.0
  - rich>=13.7.0
required_env_vars:
  - POLY_SCANNER_API_KEY (optional, for enhanced AI analysis)
  - AWS_ACCESS_KEY_ID (for AWS scans)
  - AWS_SECRET_ACCESS_KEY (for AWS scans)
  - AZURE_CLIENT_ID (for Azure scans)
  - AZURE_CLIENT_SECRET (for Azure scans)
  - GCP_CREDENTIALS_PATH (for GCP scans)
  - KUBECONFIG (for Kubernetes scans)
---

# Poly Scanner Skill

## Purpose

Poly Scanner delivers AI-augmented infrastructure analysis across multi-cloud and hybrid environments. It automatically detects security vulnerabilities, compliance drift, cost inefficiencies, and configuration anomalies in AWS, Azure, GCP, Kubernetes clusters, Docker containers, Terraform configurations, and Ansible playbooks.

Real use cases:
- Security analyst scanning AWS account for open S3 buckets and exposed EC2 instances before penetration test
- DevOps engineer validating Terraform modules against CIS AWS Foundations Benchmark before merge to main
- Cloud architect identifying over-provisioned GCP instances and idle resources to cut $50K/month spend
- Compliance officer generating PCI-DSS gap analysis report for quarterly audit
- SRE detecting configuration drift between production Kubernetes clusters and GitOps repo

## Scope

Commands available to this skill:

- `scan-aws --profile <profile> --regions <regions> --checks <check_list>`: Scan AWS account
- `scan-azure --subscription <id> --resource-groups <rgs> --tags <tag_filters>`: Scan Azure resources
- `scan-gcp --project <id> --zones <zones> --exclude <services>`: Scan GCP project
- `scan-k8s --context <context> --namespaces <ns> --include-ingress`: Scan Kubernetes cluster
- `scan-docker --host <host> --containers <container_ids> --images-only`: Scan Docker daemon
- `scan-terraform --directory <dir> --workspace <ws> --policy-type <type>`: Scan Terraform plan/state
- `scan-ansible --playbook <path> --inventory <inv> --variables <var_file>`: Scan Ansible playbook
- `analyze-report --input <file> --format <json|html|pdf> --ai-summary`: Generate AI-enhanced report
- `check-compliance --framework <cis|pci|hipaa|nist> --cloud <aws|azure|gcp>`: Validate against compliance frameworks
- `estimate-cost --savings-threshold <percent> --recommendations`: Generate cost optimization report
- `export-findings --format <csv|json|sarif> --severity <high|medium|low>`: Export scan results
- `compare-baseline --baseline <file> --current <file> --drift-threshold <percent>`: Detect configuration drift
- `monitor-continuous --webhook <url> --interval <minutes> --alert-channels <slack|email>`: Setup continuous monitoring

## Work Process

### 1. Environment Preparation

```bash
# Verify dependencies
python -c "import boto3, kubernetes, docker; print('Dependencies OK')"

# Configure AWS credentials (example)
aws configure set profile.poly-scanner.region us-east-1
aws configure set profile.poly-scanner.output json

# Set OpenClaw workspace
export OPENCLAW_WORKSPACE=/home/user/.openclaw/workspace
export POLY_SCANNER_CONFIG=$OPENCLAW_WORKSPACE/config/poly-scanner.yaml
```

### 2. Initial Discovery

```bash
# Enumerate target environment
poly-scanner discover --type aws --profile prod-account
poly-scanner discover --type k8s --context production-cluster
poly-scanner discover --type terraform --dir ./infra/
```

Expected output:
```
 discovered: 234 AWS resources across 6 services
 discovered: 89 Kubernetes pods across 3 namespaces
 discovered: 12 Terraform modules with 45 resources
```

### 3. Policy Definition

Create `$OPENCLAW_WORKSPACE/policies/poly-scanner-policy.yaml`:

```yaml
checks:
  - id: AWS_S3_BUCKET_PUBLIC_ACCESS
    severity: critical
    framework: cis
  - id: K8S_PRIVILEGED_CONTAINER
    severity: high
    framework: cis
  - id: GCP_COMPUTE_PUBLIC_IP
    severity: medium
    framework: pci
  - id: COST_IDLE_EBS_VOLUME
    severity: low
    framework: nist
exclude:
  resources:
    - "arn:aws:s3:::backup-bucket-*"
    - "k8s:namespace:monitoring"
```

### 4. Execute Scan

```bash
# Full AWS scan with AI analysis
poly-scanner scan-aws \
  --profile prod-account \
  --regions us-east-1,us-west-2 \
  --checks AWS_S3_BUCKET_PUBLIC_ACCESS,K8S_PRIVILEGED_CONTAINER \
  --policy $OPENCLAW_WORKSPACE/policies/poly-scanner-policy.yaml \
  --ai-enhance \
  --output $OPENCLAW_WORKSPACE/scans/aws-$(date +%Y%m%d-%H%M%S).json

# Kubernetes scan with namespace filtering
poly-scanner scan-k8s \
  --context prod-cluster \
  --namespaces app-prod,app-staging \
  --include-ingress \
  --exclude-resources "k8s:pod:monitoring-*" \
  --format sarif \
  > $OPENCLAW_WORKSPACE/scans/k8s-scan.sarif
```

### 5. AI-Enhanced Analysis

```bash
# Generate executive summary from findings
poly-scanner analyze-report \
  --input $OPENCLAW_WORKSPACE/scans/aws-latest.json \
  --format html \
  --ai-summary \
  --model gpt-4-turbo \
  --prompt "Summarize critical findings for CISO, prioritize by business impact, suggest 30-day remediation plan" \
  --output $OPENCLAW_WORKSPACE/reports/aws-ciso-$(date +%Y%m%d).html
```

### 6. Compliance Validation

```bash
# PCI-DSS compliance check
poly-scanner check-compliance \
  --framework pci \
  --cloud aws \
  --profile pci-env \
  --output pdf \
  --include-evidence \
  $OPENCLAW_WORKSPACE/reports/pci-compliance-$(date +%Y%m%d).pdf
```

### 7. Cost Optimization Scan

```bash
# Identify idle resources
poly-scanner estimate-cost \
  --savings-threshold 15 \
  --recommendations detailed \
  --exclude "tag:Environment=Production" \
  --output json \
  > $OPENCLAW_WORKSPACE/reports/cost-savings-$(date +%Y%m%d).json
```

### 8. Export and Integration

```bash
# Export to SARIF for CI/CD integration
poly-scanner export-findings \
  --format sarif \
  --severity high,medium \
  --output $OPENCLAW_WORKSPACE/integration/findings.sarif

# Push to monitoring webhook
curl -X POST $SLACK_WEBHOOK_URL \
  -H "Content-Type: application/json" \
  -d @$OPENCLAW_WORKSPACE/reports/alert-summary.json
```

## Golden Rules

1. **Never scan production without explicit approval**: Requires `--approved-by <email>` flag for prod environments
2. **Always use least-privilege credentials**: Scanner IAM role must have only `ReadOnlyAccess` plus custom policy for specific checks
3. **Rotate API keys quarterly**: Enforce via `POLY_SCANNER_KEY_ROTATION_DAYS=90` env var
4. **Encrypt all scan artifacts at rest**: Use `POLY_SCANNER_ENCRYPTION_KEY` for AES-256 encryption of reports
5. **Rate limit aggressively**: Default 100 requests/minute per cloud provider; override via `--throttle <rpm>`
6. **Never store credentials in reports**: Auto-redact via `--redact-secrets` flag (enforced in prod)
7. **Validate findings before action**: Use `--dry-run` for any remediation commands
8. **Baseline before scanning**: Create initial `baseline.json` with `poly-scanner compare-baseline --create`
9. **Sign all AI-generated content**: Include `X-Poly-Scanner-AI: true` header in AI-processed reports
10. **Weekly policy sync**: Pull latest policies from `git@github.com:openclaw/policies.git` via cron

## Examples

### Example 1: AWS Security Scan with AI Analysis

**Command:**
```bash
POLY_SCANNER_API_KEY=$OPENAI_KEY \
AWS_PROFILE=prod-account \
poly-scanner scan-aws \
  --regions us-east-1 \
  --checks AWS_S3_BUCKET_PUBLIC_ACCESS,AWS_RDS_PUBLIC_ACCESS,AWS_SECURITY_GROUP_OPEN_PORTS \
  --ai-enhance \
  --output-format json \
  --output /tmp/aws-scan-$(date +%s).json
```

**Input:**
```
Profile: prod-account
Regions: us-east-1
Checks: AWS_S3_BUCKET_PUBLIC_ACCESS,AWS_RDS_PUBLIC_ACCESS
AI Enhance: true
```

**Output (excerpt):**
```json
{
  "scan_id": "scan_abc123_20260302",
  "timestamp": "2026-03-02T14:32:18Z",
  "region": "us-east-1",
  "findings": [
    {
      "resource_id": "arn:aws:s3:::finance-data-bucket",
      "check_id": "AWS_S3_BUCKET_PUBLIC_ACCESS",
      "severity": "critical",
      "description": "S3 bucket has public read access via bucket policy",
      "ai_analysis": {
        "business_impact": "Critical - contains PII and financial records",
        "remediation_complexity": "low",
        "compliance_frameworks": ["PCI-DSS", "SOC2", "GDPR"],
        "exploitability": "trivial - accessible via HTTP GET",
        "generated_remediation": "aws s3api put-bucket-policy --bucket finance-data-bucket --policy file://deny-public.json"
      },
      "evidence": {
        "bucket_policy": "...",
        "public_access_block": false
      }
    }
  ],
  "summary": {
    "total": 1,
    "critical": 1,
    "high": 0,
    "medium": 0,
    "low": 0
  }
}
```

**AI-Enhanced Report:**
```
Executive Summary:
1 CRITICAL finding detected in AWS us-east-1. S3 bucket 'finance-data-bucket' 
exposes PII to public internet. Immediate action required to prevent data breach.

Business Risk:
- PCI-DSS violation: $500K potential fine
- GDPR violation: €20M or 4% global revenue
- Customer trust impact: HIGH

Remediation Plan (30 days):
Week 1: Apply bucket policy restrictions (estimated 4 hours)
Week 2: Enable S3 Block Public Access + CloudTrail monitoring
Week 3: Rotate any exposed credentials + notify affected customers
Week 4: Penetration test to verify fix
```

### Example 2: Kubernetes Compliance Scan

**Command:**
```bash
KUBECONFIG=/home/user/.kube/prod-config \
poly-scanner scan-k8s \
  --context prod-eu-west-1 \
  --namespaces app-prod,payment-prod \
  --framework cis \
  --include-ingress \
  --exclude-resources "k8s:pod:istio-proxy-*" \
  --output-format sarif \
  > /tmp/k8s-cis-scan.sarif
```

**Input:**
```
Context: prod-eu-west-1
Namespaces: app-prod, payment-prod
Framework: CIS Kubernetes Benchmark
Include Ingress: true
Exclude: istio-proxy pods
```

**Output (SARIF snippet):**
```json
{
  "version": "2.1.0",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "Poly Scanner",
          "informationUri": "https://docs.openclaw.io/poly-scanner",
          "version": "2.4.1",
          "rules": [
            {
              "id": "K8S_PRIVILEGED_CONTAINER",
              "shortDescription": {
                "text": "Privileged container detected"
              },
              "fullDescription": {
                "text": "Container running with privileged flag grants elevated host access"
              },
              "defaultConfiguration": {
                "level": "error"
              }
            }
          ]
        }
      },
      "results": [
        {
          "ruleId": "K8S_PRIVILEGED_CONTAINER",
          "level": "error",
          "message": {
            "text": "Pod 'payment-api-7dfcb8c' in namespace 'payment-prod' runs privileged container"
          },
          "locations": [
            {
              "physicalLocation": {
                "artifactLocation": {
                  "uri": "k8s://prod-eu-west-1/payment-prod/pod/payment-api-7dfcb8c"
                }
              }
            }
          ]
        }
      ]
    }
  ]
}
```

### Example 3: Terraform Drift Detection

**Command:**
```bash
poly-scanner scan-terraform \
  --directory /home/user/infra/terraform \
  --workspace production \
  --plan-file /tmp/tfplan.json \
  --policy-type owasp \
  --compare-baseline /home/user/.openclaw/workspace/baselines/infra-baseline-20260301.json \
  --drift-threshold 5 \
  --output /tmp/terraform-drift-$(date +%s).json
```

**Input:**
```
Directory: /home/user/infra/terraform
Workspace: production
Plan file: /tmp/tfplan.json
Policy: OWASP Infrastructure
Baseline: infra-baseline-20260301.json
Drift threshold: 5%
```

**Output:**
```json
{
  "scan_id": "tf_drift_xyz789",
  "baseline_date": "2026-03-01",
  "current_date": "2026-03-02",
  "drift_summary": {
    "total_resources": 156,
    "drifted_resources": 8,
    "drift_percentage": 5.13,
    "exceeds_threshold": true
  },
  "drifted_resources": [
    {
      "resource": "aws_s3_bucket.logs_bucket",
      "attribute": "lifecycle_rule.enabled",
      "baseline_value": true,
      "current_value": false,
      "change_type": "modified",
      "risk_level": "medium"
    },
    {
      "resource": "aws_security_group.web_sg",
      "attribute": "ingress.0.cidr_blocks.0",
      "baseline_value": "10.0.0.0/16",
      "current_value": "0.0.0.0/0",
      "change_type": "modified",
      "risk_level": "critical"
    }
  ],
  "recommendations": [
    "Rollback security group changes immediately - opens port 443 to world",
    "Review lifecycle rule deletion - may lose compliance logs"
  ]
}
```

### Example 4: Cost Optimization with AI Recommendations

**Command:**
```bash
poly-scanner estimate-cost \
  --cloud aws,gcp \
  --profile cost-optimization \
  --savings-threshold 20 \
  --recommendations ai \
  --exclude "tag:Environment=Production,Staging" \
  --output html \
  --ai-model gpt-4 \
  > /tmp/cost-report-$(date +%s).html
```

**AI-Generated Cost Report:**
```
Potential Monthly Savings: $47,832

1. AWS - Underutilized EC2 Instances (18% avg CPU)
   Resources: 23 instances
   Savings: $12,450/month
   Recommendation: Right-size to t3.medium or implement auto-scaling
   Risk: Low - instances have load balancer health checks

2. GCP - Idle Persistent Disks
   Resources: 47 disks (2.3TB total)
   Savings: $8,920/month
   Recommendation: Snapshot and delete; use snapshot-based disks for ephemeral workloads
   Risk: Medium - verify no stateful data before deletion

3. AWS - Unused Elastic IPs
   Resources: 12 EIPs
   Savings: $876/month
   Recommendation: Release unattached EIPs; use NAT Gateway for shared egress
   Risk: None - already unattached

4. GCP - Over-provisioned Machine Types
   Resources: 8 custom VMs with >30% memory unused
   Savings: $6,800/month
   Recommendation: Downsize to n1-standard-2 or use preemptible VMs for batch jobs
   Risk: Low - monitor performance for 2 weeks post-change

AI Summary:
Focus on EC2 right-sizing first (quick win, low risk). Address idle disks 
after implementing automated cleanup pipeline. Total feasible savings: $35K/month 
if approved changes implemented by EOY.
```

## Rollback Commands

### Rollback AWS S3 Public Access Fix
```bash
# Restore previous bucket policy (if backup created automatically)
aws s3api get-bucket-policy --bucket finance-data-bucket > /tmp/backup-policy.json
aws s3api put-bucket-policy --bucket finance-data-bucket --bucket-policy file:///tmp/backup-policy.json

# Manual rollback: re-apply original policy from S3 versioning
aws s3api get-bucket-policy-version --bucket finance-data-bucket --version-id <original_version_id> > original-policy.json
aws s3api put-bucket-policy --bucket finance-data-bucket --bucket-policy file://original-policy.json
```

### Rollback Kubernetes Privileged Container Change
```bash
# Revert pod template to previous git commit
kubectl set image deployment/payment-api payment-api=payment-api:$(git rev-parse HEAD~1) --context prod-eu-west-1

# Or restore from backup configmap
kubectl create configmap payment-api-backup --from-file=deployment.yaml=backup/deployment-v1.yaml -n payment-prod
kubectl apply -f backup/deployment-v1.yaml --context prod-eu-west-1
```

### Rollback Terraform Drift Changes
```bash
# Revert to baseline state
terraform workspace select production
terraform state pull > current-state.json
terraform state replace-provider aws aws --state=current-state.json

# Force re-apply baseline (destructive!)
terraform apply -var-file=baseline.tfvars -refresh=false -target=aws_s3_bucket.logs_bucket
```

### Rollback Cost Optimization Actions
```bash
# Restore EC2 instance types from CloudWatch metrics backup
aws ec2 modify-instance-attribute --instance-id i-0abcd1234 --instance-type "{\"Value\": \"m5.xlarge\"}"

# Re-attach and remount idle disks
gcloud compute disks attach disk-id --instance=instance-name --zone=us-east1-b
# Then manually mount and verify data integrity
```

### Complete Scan Session Rollback
```bash
# Remove all scan artifacts
rm -rf $OPENCLAW_WORKSPACE/scans/$(date +%Y%m%d)*
rm -rf $OPENCLAW_WORKSPACE/reports/$(date +%Y%m%d)*

# Reset baseline to previous day
cp $OPENCLAW_WORKSPACE/baselines/baseline-$(date -d yesterday +%Y%m%d).json \
   $OPENCLAW_WORKSPACE/baselines/current-baseline.json

# Revoke any temporary IAM permissions granted to scanner
aws iam delete-user-policy --user-name poly-scanner-temp --policy-name PolyScannerTempAccess
```

### Emergency Stop for Continuous Monitoring
```bash
# Disable continuous monitoring webhook
systemctl --user stop poly-scanner-monitor.service
# or
pkill -f "poly-scanner monitor-continuous"

# Remove webhook from all cloud event bridges
aws events remove-targets --rule PolyScannerWatch --ids PolyScannerWebhookTarget
gcloud eventarc triggers delete poly-scanner-trigger --location us-central1
```

## Verification Steps

1. **Check scan completion**:
   ```bash
   poly-scanner status --scan-id scan_abc123_20260302
   # Expected: "completed" with exit code 0
   ```

2. **Validate report integrity**:
   ```bash
   jq empty $OPENCLAW_WORKSPACE/reports/aws-ciso-20260302.json 2>/dev/null
   # Expected: valid JSON, exit 0
   ```

3. **Verify SARF format** (if CI-integrated):
   ```bash
   poly-scanner validate-sarif --input $OPENCLAW_WORKSPACE/integration/findings.sarif
   # Expected: "SARIF valid, 45 results found"
   ```

4. **Test rollback script** (in non-prod):
   ```bash
   poly-scanner rollback-test --scan-id test_scan_123 --dry-run
   # Expected: "Rollback plan generated, no changes applied"
   ```

5. **Confirm AI summary**:
   ```bash
   grep -q "X-Poly-Scanner-AI: true" $OPENCLAW_WORKSPACE/reports/*.html
   # Expected: header present in AI-enhanced reports
   ```

6. **Check encryption** (if enabled):
   ```bash
   poly-scanner verify-encryption --file $OPENCLAW_WORKSPACE/scans/aws-scan.json
   # Expected: "File encrypted with AES-256-GCM"
   ```

## Troubleshooting

### Issue: AWS credential errors
**Symptom:** `InvalidClientTokenId` or `AccessDenied` during AWS scan

**Fix:**
```bash
# Verify profile exists
aws configure list-profiles | grep poly-scanner

# Test credentials
aws sts get-caller-identity --profile poly-scanner

# Check IAM policy
aws iam get-user --user-name poly-scanner-user
aws iam list-attached-user-policies --user-name poly-scanner-user
# Ensure policy includes: ReadOnlyAccess + PolyScannerCustomPolicy
```

### Issue: Kubernetes context not found
**Symptom:** `Unable to connect to the server: context deadline exceeded`

**Fix:**
```bash
# Verify kubeconfig
echo $KUBECONFIG
ls -la $KUBECONFIG

# Test cluster access
kubectl get nodes --context prod-eu-west-1

# Refresh kubeconfig if expired
aws eks update-kubeconfig --name prod-cluster --region eu-west-1 --profile prod-account
```

### Issue: Rate limiting from cloud provider
**Symptom:** `ThrottlingException` or `429 Too Many Requests`

**Fix:**
```bash
# Reduce scan concurrency
poly-scanner scan-aws --regions us-east-1 --max-concurrent 5 --throttle 50

# Or increase via explicit retry config
cat > /tmp/retry.yaml <<EOF
retry:
  max_attempts: 5
  backoff_ms: 2000
EOF
poly-scanner scan-aws --retry-config /tmp/retry.yaml
```

### Issue: AI analysis timeout
**Symptom:** Scan completes but AI summary hangs or fails

**Fix:**
```bash
# Check API key
echo $POLY_SCANNER_API_KEY | wc -c  # Should be > 32

# Test OpenAI connectivity
curl -H "Authorization: Bearer $POLY_SCANNER_API_KEY" \
     https://api.openai.com/v1/models

# Disable AI for problematic scans
poly-scanner analyze-report --input report.json --ai-summary=false

# Or switch to local model (if installed)
poly-scanner analyze-report --input report.json --model local:phi-3.5
```

### Issue: Policy validation errors
**Symptom:** `Invalid policy: check_id not found in registry`

**Fix:**
```bash
# Update policy registry
poly-scanner policy sync --registry https://policies.openclaw.io/latest.yaml

# Verify check exists
poly-scanner policy list-checks | grep AWS_S3_BUCKET_PUBLIC_ACCESS

# Use only registered checks
poly-scanner scan-aws --checks $(poly-scanner policy list-checks --cloud aws --format csv)
```

### Issue: Disk space exhausted during scan
**Symptom:** `OSError: [Errno 28] No space left on device`

**Fix:**
```bash
# Clean old scans
find $OPENCLAW_WORKSPACE/scans -mtime +30 -delete
find $OPENCLAW_WORKSPACE/reports -mtime +90 -delete

# Redirect to larger volume
export POLY_SCANNER_TEMP_DIR=/mnt/ssd/poly-scanner-temp
mkdir -p $POLY_SCANNER_TEMP_DIR

# Stream output instead of buffering
poly-scanner scan-k8s --stream-output --output - | gzip > /mnt/ssd/k8s-scan.json.gz
```

### Issue: False positive rate too high
**Symptom:** AI analysis marks many benign findings as critical

**Fix:**
```bash
# Create suppression rules
cat > $OPENCLAW_WORKSPACE/config/poly-scanner-suppressions.yaml <<EOF
suppressions:
  - check_id: AWS_SECURITY_GROUP_OPEN_PORTS
    resource: "sg-0abcd1234"
    reason: "Port 443 open for public web access - intended"
    expires: "2026-12-31"
  - check_id: K8S_PRIVILEGED_CONTAINER
    namespace: "istio-system"
    reason: "Istio proxy requires privileged mode"
EOF

# Adjust AI temperature for more conservative analysis
poly-scanner analyze-report --input scan.json --ai-temperature 0.1
```

### Issue: Scan takes too long (>4 hours for AWS)
**Symptom:** `poly-scanner` appears stuck or extremely slow

**Fix:**
```bash
# Run in parallel mode with region segmentation
for region in us-east-1 us-east-2 us-west-1 us-west-2; do
  poly-scanner scan-aws --regions $region --output /tmp/aws-$region.json &
done
wait

# Use cached credentials (avoid repeated STS calls)
export POLY_SCANNER_CACHE_TTL=3600
poly-scanner scan-aws --use-cached-creds

# Profile scan performance
poly-scanner scan-aws --profile cpu,memory --output profile.txt
# Analyze with: snakeviz profile.txt
```
```