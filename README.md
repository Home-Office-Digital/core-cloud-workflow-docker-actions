# core-cloud-workflow-docker-actions

This repository contains actions for a full Docker workflows

- `Docker setup` - to set up docker
- `Docker build` - to build docker images
- `Docker scan` - to scan docker images using trivy
- `Docker push` - to push docker images to ECR

There are also two workflows that combine several of actions into self contained actions

- `build-scan` - will build docker images and scan them
- `build-publish` - will build, scan and push the images to ecr

## Pre-requisites

- Valid `Dockerfile` building and scanning
- Valid `AWS_ECR_ACCOUNT_ID` and `ROLE_TO_ASSUME` for pushing

## Usage

Workflows can be used as

```yml
jobs:
    build-scan:
        uses: UKHomeOffice/core-cloud-workflow-docker-actions/.github/workflows/build-scan.yml@1.1.0
        with:
            working_directory: "."
            dockerfile: "Dockerfile"
            image_name: "hello-world"
            image_tag: "latest"
            tag_latest: true

    build-scan-push:
        uses: UKHomeOffice/core-cloud-workflow-docker-actions/.github/workflows/build-scan-push.yml@1.1.0
        with:
            working_directory: "."
            dockerfile: "Dockerfile"
            image_name: "hello-world"
            image_tag: "latest"
            tag_latest: true
        secrets:
            AWS_ECR_ACCOUNT_ID: ${{ secrets.AWS_ECR_ACCOUNT_ID }}
            ROLE_TO_ASSUME: ${{ secrets.ROLE_TO_ASSUME }}
```

The actions can be used in similar manner as well