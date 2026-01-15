---
title: Python Best Practices
inclusion: fileMatch
fileMatchPattern: '**/*.py'
---

# Python Best Practices

## Code Style
- Follow PEP 8 style guide
- Use 4 spaces for indentation
- Maximum line length: 120 characters
- Use snake_case for functions and variables
- Use PascalCase for classes
- Use UPPER_SNAKE_CASE for constants

## Lambda Function Structure
- Keep handler function simple and focused
- Extract business logic into separate functions
- Use descriptive function names that explain purpose
- Include docstrings for complex functions

```python
def lambda_handler(event, context):
    """Main Lambda handler for EBS encryption configuration."""
    print(f'Received event: {event}')
    
    try:
        result = process_event(event)
        return success_response(result)
    except Exception as e:
        print(f'Error: {e}')
        return error_response(e)
```

## Boto3 Client Usage
- Create clients inside handler or functions, not at module level
- Use specific client methods rather than generic operations
- Always handle client exceptions
- Use resource-based clients for simpler operations

```python
# Good - client created in function
def enable_encryption():
    ec2_client = boto3.client('ec2')
    response = ec2_client.enable_ebs_encryption_by_default()
    return response

# Avoid - client at module level (cold start issues)
ec2_client = boto3.client('ec2')
```

## Error Handling
- Use try-except blocks for AWS API calls
- Log errors with context before re-raising
- Return meaningful error messages
- Use specific exception types when possible

```python
try:
    response = client.describe_key(KeyId=key_id)
except client.exceptions.NotFoundException:
    print(f'Key not found: {key_id}')
    # Handle missing key
except Exception as e:
    print(f'Unexpected error: {e}')
    raise
```

## Logging
- Use print() for CloudWatch Logs in Lambda
- Include context in log messages
- Log at appropriate levels (info, error)
- Use f-strings for formatted output

```python
print(f'Processing resource: {resource_id}')
print(f'Current state: {current_state}')
print(f'Error occurred: {error_message}')
```

## CloudFormation Custom Resources
- Always send response to CloudFormation (success or failure)
- Use cfnresponse helper for custom resources
- Include responseData for outputs
- Handle all request types (Create, Update, Delete)

```python
import cfnresponse

def lambda_handler(event, context):
    response_data = {}
    try:
        # Process request
        result = process_request(event)
        response_data['Result'] = result
        cfnresponse.send(event, context, cfnresponse.SUCCESS, response_data)
    except Exception as e:
        print(f'Error: {e}')
        cfnresponse.send(event, context, cfnresponse.FAILED, response_data)
```

## AWS Config Rules
- Return compliance evaluation results
- Include annotation for non-compliant resources
- Handle both configuration changes and periodic evaluations
- Use appropriate compliance types

```python
def evaluate_compliance(config_item):
    """Evaluate resource compliance against rule."""
    if meets_requirements(config_item):
        return {
            'ComplianceType': 'COMPLIANT',
            'Annotation': 'Resource meets requirements'
        }
    else:
        return {
            'ComplianceType': 'NON_COMPLIANT',
            'Annotation': 'Resource does not meet encryption requirements'
        }
```

## Dependencies
- Minimize external dependencies in Lambda functions
- Use boto3 (included in Lambda runtime)
- Package additional dependencies in deployment zip
- Document required Python version

## Testing
- Test Lambda functions locally before deployment
- Mock boto3 clients using moto or boto3 stubber
- Test all code paths (success and failure)
- Validate CloudFormation responses

## Security
- Never hardcode credentials or account IDs
- Use IAM roles for AWS service access
- Validate input from CloudFormation events
- Use KMS for encryption operations
- Follow least privilege for IAM policies
