# Build and Upload Docker Container

This GitHub Action builds a Docker container and saves it as an artifact for later deployment.

## Description

This composite action performs the following tasks:
1. Checks out the repository code
2. Sets up Docker Buildx for advanced building features
3. Builds a Docker image for Linux ARM64 platform
4. Saves the Docker image as a tar file
5. Uploads the image as a GitHub Actions artifact

The action is designed to work in CI/CD pipelines where Docker images are built once and then deployed to multiple environments.

## Inputs

| Input | Description | Required | Type |
|-------|-------------|----------|------|
| `IMAGE_NAME` | The name of the Docker image to build and save | Yes | string |

## Usage

```yaml
- name: Build Docker Container
  uses: travisfriesen/workflows/build_and_upload_docker_container@main
  with:
    IMAGE_NAME: my-app
```

## Complete Workflow Example

```yaml
name: Build and Deploy Application

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: Build and Upload Docker Container
      uses: travisfriesen/workflows/build_and_upload_docker_container@main
      with:
        IMAGE_NAME: my-application
        
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Deploy Container
      uses: travisfriesen/workflows/download_docker_container_and_run_ansible_playbook@main
      with:
        EXTRA_VARS: "app_version=${{ github.sha }}"
        ENVIRONMENT: "production"
        ARTIFACT_NAME: "my-application"
        USER: "deploy"
        PLAYBOOK_PATH: "./ansible/deploy.yml"
        HOST: "prod-server.example.com"
```

## Technical Details

- **Platform**: Builds for `linux/arm64` architecture
- **Caching**: Uses GitHub Actions cache for faster builds
- **Artifact retention**: 14 days (configurable in the action)
- **Output format**: Docker image saved as `.tar` file

## Prerequisites

- A `Dockerfile` must exist in the repository root
- The repository must be accessible for checkout

## Dependencies

- `actions/checkout@v4` - Repository checkout
- `docker/setup-buildx-action@v3` - Docker Buildx setup
- `docker/build-push-action@v5` - Docker image building
- `actions/upload-artifact@v4` - Artifact upload

## Output Artifacts

The action creates an artifact with:
- **Name**: The value of `IMAGE_NAME` input
- **Content**: A tar file containing the Docker image (`{IMAGE_NAME}.tar`)
- **Retention**: 14 days

## Notes

- The Docker image is tagged as `{IMAGE_NAME}:latest`
- GitHub Actions cache is used to speed up subsequent builds
- The image is built but not pushed to any registry
