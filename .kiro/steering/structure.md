# Project Structure

## Root Directory

```
cloud-solution-library/
├── docs/                           # Documentation content (Markdown)
├── templates/                      # AWS Service Catalog CloudFormation templates
├── techdocs-publisher/             # TechDocs ingestion infrastructure
├── venv/                          # Python virtual environment (gitignored)
├── mkdocs.yml                     # MkDocs config for local development
├── mkdocs-pages.yml               # MkDocs config for GitLab Pages deployment
├── requirements.txt               # Python dependencies
├── catalog-info.yaml              # Backstage entity definition
└── .gitlab-ci.yml                 # CI/CD pipeline configuration
```

## Documentation Structure (`docs/`)

Organized by user journey and feature area:

```
docs/
├── index.md                       # Homepage
├── support.md                     # Support contact information
├── contributing.md                # Contribution guidelines
├── quickstart/                    # Getting started guides
│   ├── overview.md
│   ├── prerequisites.md
│   ├── access-management.md
│   └── account-setup/             # Multi-step account setup process
│       ├── index.md               # Setup overview
│       ├── request.md             # Step 1: Request
│       ├── intake-meeting.md      # Step 2: Intake meeting
│       ├── provisioning.md        # Step 3: Provisioning
│       ├── handover.md            # Step 4: Handover
│       └── troubleshooting.md     # Common issues
└── service-catalog/               # Service Catalog product documentation
    ├── index.md
    ├── cdk-bootstrap/             # CDK Bootstrap product
    │   ├── index.md
    │   ├── quick-start.md
    │   └── examples.md
    └── gitlab-oidc/               # GitLab OIDC product
        ├── index.md
        ├── quick-start.md
        ├── architecture.md
        ├── examples.md
        ├── best-practices.md
        ├── troubleshooting.md
        ├── faq.md
        └── release-notes.md
```

### Documentation Conventions

- Each feature has an `index.md` as the landing page
- Multi-step processes use numbered prefixes (1., 2., 3.) in navigation
- Troubleshooting and FAQ pages are separate from main content
- Architecture diagrams use Mermaid syntax

## Service Catalog Templates (`templates/`)

CloudFormation templates for AWS Service Catalog products:

```
templates/
├── sc-gitlab-oidc.yaml                              # GitLab OIDC integration
├── sc-gitlab-oidc-hub-tooling.yaml                  # Hub account variant
├── sc-gitlab-oidc-spoke-remote-deploy-role.yaml     # Spoke account variant
└── sc-cdk-bootstrap-with-permission-boundaries.yaml # CDK Bootstrap with boundaries
```

### Template Naming Convention

- Prefix: `sc-` (Service Catalog)
- Kebab-case naming
- Descriptive of the resource/feature

## TechDocs Publisher (`techdocs-publisher/`)

Infrastructure for publishing documentation to Backstage:

```
techdocs-publisher/
├── cdk/                           # CDK infrastructure code
│   ├── bin/                       # CDK app entry point
│   ├── lib/                       # Stack and construct definitions
│   │   ├── TechdocsIngestionStack.ts
│   │   ├── EnEnTechdocsS3Upload.ts
│   │   └── techdocs-s3-upload-lambda/  # Lambda handler code
│   │       ├── index.ts
│   │       └── index.test.ts
│   ├── tags/                      # Tagging utilities
│   ├── package.json
│   ├── tsconfig.json
│   └── cdk.json
├── vale/                          # Documentation linting
│   ├── .vale.ini                  # Vale configuration
│   └── styles/                    # Custom style rules
│       ├── EnEn/                  # ANWB style guide rules
│       └── config/                # Filters, scripts, templates
├── scripts/                       # Build and deployment scripts
│   ├── build-and-deploy-docs.sh
│   └── lint-docs.sh
├── templates/                     # GitLab CI templates
│   └── publish-techdocs.yml       # Reusable CI job
├── openapi/                       # API specifications
│   └── doc-ingestor.openapi.yaml
├── Dockerfile                     # Container for TechDocs generation
└── README.md                      # Usage documentation
```

### CDK Structure Conventions

- **bin/**: Application entry points
- **lib/**: Stack and construct implementations
- **lib/[construct-name]-lambda/**: Lambda handlers co-located with constructs
- **tags/**: Cross-cutting concerns like tagging
- Tests co-located with source files (`.test.ts` suffix)

### Vale Linting Structure

- **styles/EnEn/**: Custom rules for ANWB documentation standards
  - British English enforcement
  - Inclusive language checks
  - Consistency rules (naming, abbreviations)
  - Heading structure validation
- **styles/config/**: Advanced validation logic
  - Tengo scripts for complex rules
  - Expression filters
  - Custom templates

## Configuration Files

### Root Level

- `.gitlab-ci.yml`: Pipeline definition with stages and jobs
- `mkdocs.yml`: Minimal config compatible with techdocs-cli
- `mkdocs-pages.yml`: Enhanced config with Material theme features
- `catalog-info.yaml`: Backstage entity metadata
- `requirements.txt`: Python dependencies for MkDocs

### CDK Level

- `techdocs-publisher/cdk/cdk.json`: CDK app configuration
- `techdocs-publisher/cdk/tsconfig.json`: TypeScript compiler settings
- `techdocs-publisher/cdk/jest.config.json`: Jest test configuration
- `techdocs-publisher/cdk/.prettierrc.yaml`: Code formatting rules

## Naming Conventions

### Files

- **Markdown**: Kebab-case (e.g., `quick-start.md`, `intake-meeting.md`)
- **TypeScript**: PascalCase for classes/constructs (e.g., `TechdocsIngestionStack.ts`)
- **Scripts**: Kebab-case with `.sh` extension
- **Templates**: Kebab-case with descriptive names

### Directories

- Kebab-case for all directories
- Plural for collections (e.g., `docs/`, `templates/`, `scripts/`)
- Singular for specific features (e.g., `cdk/`, `vale/`)

## Build Artifacts

Generated files (gitignored):

- `site/`: Built MkDocs static site
- `public/`: GitLab Pages deployment directory
- `venv/`: Python virtual environment
- `techdocs-publisher/cdk/node_modules/`: Node.js dependencies
- `techdocs-publisher/cdk/cdk.out/`: CDK synthesis output
