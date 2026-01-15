---
title: Testing Best Practices
inclusion: always
---

# Testing Best Practices

## Python Testing
- Use pytest for Python Lambda functions
- Run tests quietly: `pytest -q` or `python -m pytest --tb=short -q`
- Filter specific tests: `pytest -k "test_specific"`
- Stop on first failure: `pytest --maxfail=1`

## Test Organization
- Keep test files alongside Lambda function code
- Use descriptive test names that explain what's being tested
- Mock AWS service calls using moto or boto3 stubber
- Separate unit tests from integration tests

## CI/CD Testing
- Run tests with minimal output in GitLab CI
- Capture test results as artifacts
- Use pytest-cov for coverage reporting
- Fail builds on test failures