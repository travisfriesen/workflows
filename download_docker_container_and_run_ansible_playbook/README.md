# Download Docker Container and Run Ansible Playbook

This GitHub Action downloads a Docker container artifact and runs an Ansible playbook to deploy it to a target host.

## Description

This composite action performs two main tasks:
1. Downloads a Docker image artifact from GitHub Actions artifacts
2. Executes an Ansible playbook to deploy the downloaded container

This action is typically used in CI/CD pipelines where Docker images are built in one job, uploaded as artifacts, and then deployed to target environments using Ansible automation.

## Inputs

| Input | Description | Required | Type |
|-------|-------------|----------|------|
| `EXTRA_VARS` | Extra variables for the Ansible playbook | Yes | string |
| `ENVIRONMENT` | The deployment environment (e.g., staging, production) | Yes | string |
| `ARTIFACT_NAME` | The name of the Docker image artifact to download | Yes | string |
| `USER` | The user to run the Ansible playbook as | Yes | string |
| `PLAYBOOK_PATH` | The path to the Ansible playbook to run | Yes | string |
| `HOST` | The host to run the Ansible playbook against | Yes | string |

## Usage

```yaml
- name: Deploy Docker Container
  uses: travisfriesen/workflows/download_docker_container_and_run_ansible_playbook@main
  with:
    EXTRA_VARS: "container_name=my-app version=1.0.0"
    ENVIRONMENT: "production"
    ARTIFACT_NAME: "my-app-docker-image"
    USER: "deploy"
    PLAYBOOK_PATH: "./ansible/deploy.yml"
    HOST: "192.168.1.100"
```

   ## GitHub Environment Deployment

To deploy to a GitHub environment with approval workflows and environment-specific secrets:

```yaml
name: Deploy Application

on:
  workflow_run:
    workflows: ["Build and Upload Docker Container"]
    types:
      - completed

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Deploy to Staging
      uses: travisfriesen/workflows/download_docker_container_and_run_ansible_playbook@main
      with:
        EXTRA_VARS: "app_version=${{ github.sha }} environment=staging"
        ENVIRONMENT: "staging"
        ARTIFACT_NAME: "my-application"
        USER: "${{ secrets.ANSIBLE_USER }}"
        PLAYBOOK_PATH: "./ansible/deploy.yml"
        HOST: "${{ vars.STAGING_HOST }}"

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Deploy to Production
      uses: travisfriesen/workflows/download_docker_container_and_run_ansible_playbook@main
      with:
        EXTRA_VARS: "app_version=${{ github.sha }} environment=production"
        ENVIRONMENT: "production"
        ARTIFACT_NAME: "my-application"
        USER: "${{ secrets.ANSIBLE_USER }}"
        PLAYBOOK_PATH: "./ansible/deploy.yml"
        HOST: "${{ vars.PRODUCTION_HOST }}"
```

## Multi-Environment Deployment with Matrix

For deploying to multiple environments:

```yaml
name: Deploy to Multiple Environments

on:
  workflow_run:
    workflows: ["Build and Upload Docker Container"]
    types:
      - completed

jobs:
  deploy:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    strategy:
      matrix:
        environment: [staging, production]
        include:
          - environment: staging
            host: staging-server.example.com
            requires_approval: false
          - environment: production
            host: prod-server.example.com
            requires_approval: true
    
    environment: 
      name: ${{ matrix.environment }}
      url: https://${{ matrix.host }}
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Deploy to ${{ matrix.environment }}
      uses: travisfriesen/workflows/download_docker_container_and_run_ansible_playbook@main
      with:
        EXTRA_VARS: "app_version=${{ github.sha }} environment=${{ matrix.environment }}"
        ENVIRONMENT: "${{ matrix.environment }}"
        ARTIFACT_NAME: "my-application"
        USER: "${{ secrets.ANSIBLE_USER }}"
        PLAYBOOK_PATH: "./ansible/deploy.yml"
        HOST: "${{ matrix.host }}"
```

## Environment Configuration

### Setting up GitHub Environments

1. Go to your repository → Settings → Environments
2. Create environments (e.g., `staging`, `production`)
3. Configure environment-specific settings:
   - **Protection rules**: Require reviewers for production
   - **Environment secrets**: Store sensitive data like SSH keys
   - **Environment variables**: Store non-sensitive config like hostnames

### Environment Secrets and Variables

- **Secrets** (sensitive): `ANSIBLE_USER`, `SSH_PRIVATE_KEY`, `VAULT_PASSWORD`
- **Variables** (non-sensitive): `STAGING_HOST`, `PRODUCTION_HOST`, `APP_PORT`

## Prerequisites

- The Docker image artifact must be available in the current workflow run or accessible workflow artifacts
- Ansible must be installed on the runner
- The target host must be accessible and configured for Ansible connections
- SSH keys or other authentication methods must be properly configured for the specified user
- GitHub environments must be configured if using environment-specific deployments

## Notes

- The action sets `ANSIBLE_HOST_KEY_CHECKING=False` to disable SSH host key checking
- The downloaded artifact is stored in `/tmp/${{ inputs.ENVIRONMENT }}/`
- The Ansible inventory uses a single host format: `${{ inputs.HOST }},`
- Use `environment:` at the job level to enable GitHub environment features

## Dependencies

- `actions/download-artifact@v4` - Used to download the Docker image artifact

## Security Considerations

- Ensure the `USER` input has appropriate permissions on the target host
- Consider using SSH keys or other secure authentication methods
- Be cautious with the `EXTRA_VARS` input as it may contain sensitive information
- Host key checking is disabled, which may pose security risks in some environments
- Use GitHub environment protection rules for production deployments
- Store sensitive configuration in environment secrets, not repository secrets
