# Build and Upload Docker Container Action

This GitHub Action builds a Docker container with support for custom build arguments and saves it as an artifact for later deployment.

## Features

- Builds Docker images for ARM64 architecture
- Supports custom build arguments
- Uses GitHub Actions cache for faster builds
- Saves the built image as an artifact with 14-day retention
- Creates multiple tags (latest and run-specific)

## Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `IMAGE_NAME` | The name of the Docker image to build and save | Yes      |
| `BUILD_ARGS` | The build arguments for Docker to build | No       |

## Usage

```yaml
- name: Build Docker Container
  uses: ./build_and_upload_docker_container
  with:
    IMAGE_NAME: my-app
    BUILD_ARGS: |
      ARG1=value1
      ARG2=value2
      NODE_ENV=production
```

## Build Arguments

This action supports Docker build arguments through the `BUILD_ARGS` input. You can pass multiple build arguments in the format:

```yaml
BUILD_ARGS: |
  ARG_NAME1=value1
  ARG_NAME2=value2
  ENVIRONMENT=production
```

These build arguments will be passed to the Docker build process and can be used in your Dockerfile with the `ARG` instruction.

## Output

The action will:
1. Build the Docker image with the specified name and build arguments
2. Tag the image with both `latest` and `{run_number}-{run_attempt}` tags
3. Save the image as a tar file
4. Upload it as a GitHub Actions artifact

The artifact will be named after your `IMAGE_NAME` and will be available for download for 14 days.
