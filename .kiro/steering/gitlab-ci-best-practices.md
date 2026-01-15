---
title: GitLab CI Best Practices
inclusion: fileMatch
fileMatchPattern: '.gitlab-ci.yml'
---

# GitLab CI Best Practices

## Pipeline Structure
- Use clear stage names (test, deploy)
- Separate test and deployment stages
- Use `only` and `except` for branch control
- Specify appropriate Docker images

```yaml
stages:
  - test
  - deploy

test:
  stage: test
  except:
    - master
  script:
    - python --version
    - pytest -q

deploy:
  stage: deploy
  only:
    - master
  script:
    - python gitlab-ci/upload_to_s3.py
```

## Python Environment
- Use specific Python version (python:3.12)
- Install dependencies at start of job
- Use pip for package installation
- Cache dependencies when possible

```yaml
deploy:
  image: "python:3.12"
  before_script:
    - pip install boto3
  script:
    - python gitlab-ci/upload_to_s3.py
```

## AWS Credentials
- Use GitLab CI/CD variables for AWS credentials
- Never commit credentials to repository
- Use IAM roles when possible
- Rotate credentials regularly
- Set variables as protected and masked

## Deployment Scripts
- Keep deployment scripts in `gitlab-ci/` directory
- Use descriptive script names
- Include error handling in scripts
- Log deployment progress
- Validate uploads to S3

## S3 Upload Patterns
- Upload configuration zip to configuration bucket
- Upload templates to nested templates bucket
- Upload Lambda functions to Lambda bucket
- Use KMS encryption for sensitive data
- Verify uploads completed successfully

```python
# Configuration upload
s3.upload_file(
    'config.zip',
    'custom-control-tower-configuration-<account>-<region>',
    'config.zip',
    ExtraArgs={
        'ServerSideEncryption': 'aws:kms',
        'SSEKMSKeyId': kms_key_id
    }
)
```

## Lambda Packaging
- Package each Lambda function separately
- Include all dependencies in zip
- Use consistent naming: `function.zip` or `<function-name>.zip`
- Preserve directory structure for imports
- Test packaged functions before upload

## Error Handling
- Exit with non-zero code on failure
- Log errors with context
- Use try-except in Python scripts
- Validate prerequisites before deployment
- Provide clear error messages

```python
try:
    s3.upload_file(file_path, bucket, key)
    print(f'Successfully uploaded {key}')
except Exception as e:
    print(f'Failed to upload {key}: {e}')
    sys.exit(1)
```

## Testing
- Run validation in test stage
- Test Python syntax before deployment
- Validate CloudFormation templates
- Check manifest.yaml syntax
- Run unit tests for Lambda functions

## Deployment Validation
- Verify S3 uploads completed
- Check CodePipeline triggered
- Monitor StackSet operations
- Validate resources in target accounts
- Review CloudWatch Logs for errors

## Branch Strategy
- Deploy only from master/main branch
- Test on feature branches
- Use merge requests for changes
- Require approvals for production changes
- Tag releases appropriately

## Notifications
- Configure pipeline failure notifications
- Alert on deployment failures
- Notify teams of SCP changes
- Document deployment windows
- Communicate breaking changes

## Performance
- Minimize pipeline execution time
- Cache dependencies between jobs
- Parallelize independent jobs
- Use artifacts for inter-job data transfer
- Clean up temporary files

## Security
- Scan for secrets before commit
- Use protected branches
- Require signed commits for production
- Audit pipeline execution logs
- Restrict pipeline variable access
- Use least privilege for AWS credentials

## Documentation
- Document pipeline stages in README
- Explain deployment process
- List required GitLab variables
- Document rollback procedures
- Maintain runbook for failures
