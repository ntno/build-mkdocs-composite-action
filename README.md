# build-mkdocs-composite-action
## features
- configures AWS credentials
- sets up python and loads pip cache
- builds mkdocs site
- archives mkdocs site to AWS (optional)

## usage

### [Recommended - Assume Role directly using GitHub OIDC provider](https://github.com/aws-actions/configure-aws-credentials#assuming-a-role)
*__note:__* using OIDC requires `id-token` write and `contents` read  permissions granted to GITHUB_TOKEN. see [Assigning permissions](https://docs.github.com/en/actions/using-jobs/assigning-permissions-to-jobs) for more information on assigning workflow permissions.
```
permissions:
  id-token: write
  contents: read
jobs:
  build-pr:
    runs-on: ubuntu-latest
    steps: 
      - name: Checkout
        uses: actions/checkout@v3
      - name: Build PR
        uses: ntno/build-mkdocs-composite-action@v3
        with:
          archive-enabled: true
          version: 1.0.3-pr
          env-name: prod
          aws-region: us-east-2
          role-to-assume: arn:aws:iam::************:role/CI-ntno.net
```

### [IAM User](https://github.com/aws-actions/configure-aws-credentials#assuming-a-role)
*__note:__* Secrets for composite actions must be configured using an [`environment`](https://docs.github.com/en/actions/using-jobs/using-environments-for-jobs).  The `ci` environment is used in the following example.  It contains the secrets `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.

```
jobs:
  build-pr:
    runs-on: ubuntu-latest
    environment: ci
    steps: 
      - name: Checkout
        uses: actions/checkout@v3
      - name: Build PR
        uses: ntno/build-mkdocs-composite-action@v3
        with:
          archive-enabled: true
          version: 1.0.3-pr
          env-name: prod
          aws-region: us-east-2
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## inputs
```
  archive-enabled:
    description: 'Enables archiving of build artifact.  Ex: `true` or `false`'
    default: false
    required: true
    type: boolean
  version:
    description: 'Build version.  Used to store archive at unique S3 key.  Ex: sha-####, 3.0.0'
    required: false
    type: string
  env-name:
    description: 'Environment name.  Ex: dev, prod'
    default: 'dev'
    required: true
    type: string
  make-vars:
    description: 'Variables to pass to all make commands.  Ex: `QUIET=1` would result in `make QUIET=1 build-mkdocs...`'
    default: "--no-print-directory"
    required: false
    type: string
  aws-region:
    description: 'AWS Region.  Ex: us-east-1'
    default: 'us-east-1'
    required: true
    type: string                      
  aws-access-key-id:
    description: 'AWS Access Key ID.'
    required: false
  aws-secret-access-key:
    description: 'AWS Secret Access Key.'
    required: false
  role-to-assume:
    description: >-
      Use the provided credentials to assume an IAM role and configure the Actions
      environment with the assumed role credentials rather than with the provided
      credentials
    required: false
  role-duration-seconds:
    description: "Role duration in seconds (default: 6 hours, 1 hour for OIDC/specified aws-session-token)"
    required: false
  role-session-name:
    description: 'Role session name (default: GitHubActions)'
    required: false
  role-external-id:
    description: 'The external ID of the role to assume'
    required: false
  role-skip-session-tagging:
    description: 'Skip session tagging during role assumption'
    required: false    
  python-version:
    description: 'Python Version'
    default: '3.8'
    required: true
    type: string
```

## prerequisites
expects Makefile in mkdocs project with the following directives:
```
setup-mkdocs: env region download-directory
build-mkdocs: env region
archive-mkdocs: env region version
```
### setup-mkdocs
use this directive to handle any steps prior to building (ex: download external image assets)

### build-mkdocs
use this directive to build mkdocs site

### archive-mkdocs (optional)
use this directive to upload the site to S3

## see also  
- [Use OpenID Connect within your workflows to authenticate with Amazon Web Services.](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services) 
- [AWS Credentials GitHub action](https://github.com/aws-actions/configure-aws-credentials)

## references
- https://docs.github.com/en/actions/creating-actions/creating-a-composite-action
- https://docs.github.com/en/actions/using-jobs/using-environments-for-jobs