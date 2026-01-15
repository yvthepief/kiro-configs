---
title: AWS Control Tower Customizations (CfCT) Best Practices
inclusion: always
---

# AWS Control Tower Customizations (CfCT) Best Practices

## Manifest Structure
- Use version: `2021-03-15` for current CfCT
- Set region to home region: `eu-central-1`
- Disable stack set deletion: `enable_stack_set_deletion: false`
- Organize resources logically (SCPs first, then StackSets)

```yaml
region: eu-central-1
version: 2021-03-15
enable_stack_set_deletion: false

resources:
  # SCPs
  - name: scp-name
    ...
  
  # StackSets
  - name: stackset-name
    ...
```

## Resource Naming Conventions
- SCPs: `aws-landing-zone-<ou>-<purpose>-scp`
- Region SCPs: `aws-landing-zone-<ou>-<purpose>-region-scp`
- StackSets: PascalCase describing purpose (e.g., `KMSKey`, `IdentityProviders`)
- Templates: `CustomControlTower-<Purpose>.template`
- Lambda functions: `<category>-<purpose>` (e.g., `cfn-ebs-default-encryption`)

## Service Control Policies (SCPs)

### Deployment
- Place policy files in `policies/` directory
- Use descriptive names matching resource name
- Test in sandbox OU first
- Communicate changes to all teams before deployment
- Document policy purpose in description

```yaml
- name: aws-landing-zone-internal-prod-scp
  description: To prevent ANWB prohibited activities in prod internal accounts
  resource_file: policies/aws-landing-zone-internal-prod-scp.json
  deploy_method: scp
  deployment_targets:
    organizational_units:
      - anwb-landing-zone-internal:Prod
```

### Policy Structure
- Use explicit Deny statements for restrictions
- Include conditions for exceptions
- Document each statement purpose
- Test with AWS Policy Simulator
- Consider impact on all services

### Region Restrictions
- Deploy separate region SCPs
- Allow eu-central-1 and eu-west-1
- Allow us-east-1 for global services only
- Use NotAction for global service exceptions

## StackSets

### Deployment Targets
- Use OU targeting for consistent deployments
- Use account targeting for unique configurations
- List OUs in logical order (Dev, PreProd, Prod)
- Document why specific accounts are targeted

```yaml
deployment_targets:
  organizational_units:
    - anwb-landing-zone-internal:Dev
    - anwb-landing-zone-internal:PreProd
    - anwb-landing-zone-internal:Prod
  accounts:
    - aws-anwb-itops-log-archive
```

### Parameters
- Use parameters for environment-specific values
- Avoid hardcoding account IDs or regions
- Use SSM parameters for cross-stack references
- Document parameter purposes

```yaml
parameters:
  - parameter_key: "KmsKeyId"
    parameter_value: "alias/aws/ebs"
  - parameter_key: "Environment"
    parameter_value: "Production"
```

### Regions
- Default to home region (eu-central-1) if not specified
- Specify regions explicitly for multi-region resources
- Deploy to us-east-1 only when required for global services

```yaml
regions:
  - eu-central-1
  - eu-west-1
```

## SSM Parameter Exports
- Use consistent naming: `/org/member/<account>/<resource>`
- Export resource identifiers for cross-account access
- Document parameter dependencies
- Use in dependent StackSets

```yaml
export_outputs:
  - name: /org/member/logging/bucket_name
    value: $[output_BucketName]
  - name: /org/member/sharedservices/vpc_id
    value: $[output_VPCID]
```

## Lambda Functions

### Organization
- Group by category: `cfn-*`, `config-rules-*`, `security-fixes-*`
- One function per directory
- Include all dependencies in directory
- Use consistent handler names

### Packaging
- GitLab CI automatically packages functions
- Creates zip files for each function
- Uploads to Lambda functions bucket
- Preserves directory structure

### CloudFormation Integration
- Reference Lambda from S3 in templates
- Use consistent S3 key structure
- Include execution role in template
- Configure appropriate timeout and memory

## Deployment Workflow

### Making Changes
1. Update manifest.yaml or templates locally
2. Test in sandbox or dev OU first
3. Commit and push to feature branch
4. Create merge request for review
5. Merge to master to trigger deployment
6. Monitor CodePipeline execution
7. Verify StackSet operations
8. Validate resources in target accounts

### Testing
- Always test in non-production first
- Validate CloudFormation templates
- Check for syntax errors in manifest
- Review SCP impact before deployment
- Test Lambda functions locally

### Rollback
- Revert Git commit if issues occur
- Delete failed StackSet instances
- Detach problematic SCPs via console
- Document rollback procedures
- Communicate issues to teams

## Critical Resources

### Deployment Order
1. S3 access log buckets (required first)
2. KMS keys
3. Identity providers
4. IAM boundaries and roles
5. Network resources
6. Application-specific resources

### Dependencies
- S3 buckets before resources that log to them
- KMS keys before encrypted resources
- IAM roles before resources that use them
- VPCs before resources that deploy into them

## Organizational Units

### Naming Convention
- Use hierarchical structure: `anwb-landing-zone-<category>:<environment>`
- Categories: internal, public, hybrid, security, connectivity, sandbox
- Environments: Dev, PreProd, Prod, Non-prod

### Targeting
- Target entire OU for consistent policies
- Use environment-specific SCPs
- Deploy baseline resources to all OUs
- Use account targeting for exceptions

## Security Considerations

### Communication
- Inform all teams before SCP changes
- Document impact of policy changes
- Provide advance notice for breaking changes
- Maintain change log

### Validation
- Test SCPs in sandbox first
- Verify no unintended denials
- Check CloudTrail for denied actions
- Use AWS Policy Simulator

### Monitoring
- Monitor StackSet operation status
- Review CloudWatch Logs for Lambda errors
- Check Config rule compliance
- Track SCP effectiveness

## Troubleshooting

### Common Issues
- StackSet deployment failures: Check IAM permissions and resource limits
- SCP conflicts: Review all policies at OU hierarchy
- Lambda errors: Check CloudWatch Logs and execution role
- Parameter not found: Verify SSM parameter exports completed

### Debugging
- Check CodePipeline execution logs
- Review Step Functions state machine
- Examine CloudFormation stack events
- Validate manifest.yaml syntax

## Documentation
- Document all custom resources in README
- Explain SCP purposes and restrictions
- List Lambda function categories
- Maintain architecture diagrams
- Update documentation with changes
