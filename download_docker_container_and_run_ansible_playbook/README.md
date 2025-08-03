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

## Example Workflow

```yaml
name: Deploy Application

on:
  workflow_run:
    workflows: ["Build and Upload Docker Container"]
    types:
      - completed

jobs:
  deploy:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Deploy to Production
      uses: travisfriesen/workflows/download_docker_container_and_run_ansible_playbook@main
      with:
        EXTRA_VARS: "app_version=${{ github.sha }} environment=production"
        ENVIRONMENT: "production"
        ARTIFACT_NAME: "production-docker-image"
        USER: "ansible"
        PLAYBOOK_PATH: "./playbooks/deploy-app.yml"
        HOST: "prod-server.example.com"
```

## Prerequisites

- The Docker image artifact must be available in the current workflow run or accessible workflow artifacts
- Ansible must be installed on the runner
- The target host must be accessible and configured for Ansible connections
- SSH keys or other authentication methods must be properly configured for the specified user

## Notes

- The action sets `ANSIBLE_HOST_KEY_CHECKING=False` to disable SSH host key checking
- The downloaded artifact is stored in `/tmp/${{ inputs.ENVIRONMENT }}/`
- The Ansible inventory uses a single host format: `${{ inputs.HOST }},`

## Dependencies

- `actions/download-artifact@v4` - Used to download the Docker image artifact

## Security Considerations

- Ensure the `USER` input has appropriate permissions on the target host
- Consider using SSH keys or other secure authentication methods
- Be cautious with the `EXTRA_VARS` input as it may contain sensitive information
- Host key checking is disabled, which may pose security risks in some environments
