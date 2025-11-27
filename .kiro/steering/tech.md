# Technology Stack

## Documentation Platform

- **MkDocs**: Static site generator (v1.5.0+)
- **Material for MkDocs**: Theme (v9.0.0+)
- **mkdocs-techdocs-core**: Backstage TechDocs integration (v1.0.0+)
- **Python**: 3.8+ required for MkDocs tooling

### MkDocs Plugins & Extensions

- `pymdownx.superfences`: Code blocks and Mermaid diagram support
- `mkdocs-mermaid2-plugin`: Mermaid diagram rendering
- `admonition`: Callout boxes
- `toc`: Table of contents with permalinks

## TechDocs Publisher Infrastructure

### CDK Stack (TypeScript)

- **AWS CDK**: v2.199.0
- **TypeScript**: v5.8.0
- **Node.js**: Compatible with Node 22.x
- **Testing**: Jest v30.0.0 with ts-jest
- **Build**: esbuild v0.25.4

### Lambda Functions

- **Runtime**: Node.js (via NodejsFunction construct)
- **Backstage Integration**: @backstage/plugin-techdocs-node v1.13.3
- **Catalog Model**: @backstage/catalog-model v1.7.4

### Key Dependencies

- `cdk-nag`: v2.36.14 (security/compliance checks)
- `adm-zip`: v0.5.16 (artifact packaging)
- `@smithy/node-http-handler`: v4.0.6

## Service Catalog Templates

- **CloudFormation**: YAML templates for AWS Service Catalog
- **GitLab OIDC**: Identity federation for CI/CD
- **CDK Bootstrap**: Custom bootstrap with permission boundaries

## Documentation Linting

- **Vale**: Style and grammar linting
- **Custom Rules**: EnEn style guide (British English, inclusive language, consistency)
- **Filters**: Tengo scripts for advanced validation

## Common Commands

### Local Development

```bash
# Setup virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Serve documentation locally (recommended)
npx techdocs-cli serve

# Alternative: Direct MkDocs
mkdocs serve
```

### Building

```bash
# Local build
mkdocs build

# GitLab Pages build (with enhanced features)
mkdocs build --config-file mkdocs-pages.yml
```

### CDK Operations (techdocs-publisher)

```bash
cd techdocs-publisher/cdk

# Install dependencies
npm install

# Build TypeScript
npm run build

# Run tests
npm test

# CDK commands
npm run cdk -- synth
npm run cdk -- deploy
```

### Testing

```bash
# CDK tests (from techdocs-publisher/cdk)
npm test

# Lambda function tests
npm test -- --testPathPattern=lambda
```

### Linting Documentation

```bash
# Run Vale linting
cd techdocs-publisher
./scripts/lint-docs.sh
```

## CI/CD Pipeline

- **GitLab CI**: `.gitlab-ci.yml` orchestrates all stages
- **Stages**: Test → Build → Deploy → Publish-techdocs
- **Artifacts**: Built site stored for 1 hour
- **Deployment**: Manual approval required for both GitLab Pages and TechDocs
- **Cache**: pip cache for faster builds

## Configuration Files

- `mkdocs.yml`: Basic config for local development with techdocs-cli
- `mkdocs-pages.yml`: Enhanced config with Material theme for GitLab Pages
- `catalog-info.yaml`: Backstage entity definition
- `techdocs-publisher/cdk/cdk.json`: CDK app configuration
