# action-buildnpush

GitHub Action for build and push docker images, Sysdig way

## Inputs

### Required

- `image_name`: name of the docker image. Example: redis-6
- `image_description`: "description of the docker image. Example: sysdig redis-6 image"
- `context_path`: path to docker directory (where Dockerfile is)
- `github_token`: "Github Token used to tag the repo"

## Optional

- `dockerfile`:  path to docker file (defaults to {{context_path}}/Dockerfile)
- `dry_run`: whether to actually create git tag and push the image (default: false)
- `push_to_artifactory`: whether push image to artifactory (default: true)
- `push_to_quay`: whether push image to quay.io (default: false)
- `artifactory_prefix`: prefix for artifactory repository. Default: `docker.internal.sysdig.com`
- `quay_prefix`: prefix for quay repository. Default: `quay.io/sysdig`

- `artifactory_username`: "Artifactory Username Secret (Needed only if `push_to_artifactory` is `true`) "
- `artifactory_password`: "Artifactory Password Secret (Needed only if `push_to_artifactory` is `true`)"
- `artifactory_url`: Artifactory URL

- `quay_username`: "Quay Username Secret (Needed only if `push_to_quay` is `true`)"
- `quay_password`: "Quay Password Secret (Needed only if `push_to_quay` is `true`)"


## Outputs

- `names`: Image name(s) generated
- `tags`: calculate git tag
- `labels`: Labels added to container image(s)


## Example workflow

Perform all checks on pull requests

```yaml
name: Build And Push

on:
  # allows to manually run the workflow from the UI
  workflow_dispatch:
    inputs:
      dry_run:
        description: 'DryRun'
        required: true
        default: 'false'


  pull_request:
    types: [opened, edited, synchronize, reopened]
    paths:
      - 'containers/redis/**'

  push:
    branches: ["main"]
    tags: ["*"]
    paths:
      - 'containers/**'

jobs:
  build:
    runs-on: self-hosted
    steps:
    - uses: actions/checkout@v2
    - uses: draios/infra-action-buildnpush@v1
      with:
        push_to_quay: true
        image_name: "redis-6"
        image_description: "sysdig image for redis-6"
        context_path: "containers/redis"
        dockerfile: "Dockerfile"
        dry_run: ${{ ! ((github.event_name == 'push' && github.ref == 'refs/heads/main') || (github.event_name == 'workflow_dispatch' && github.event.inputs.dry_run == 'false')) }}
        artifactory_username: ${{ secrets.ARTIFACTORY_USERNAME }}
        artifactory_password: ${{ secrets.ARTIFACTORY_PASSWORD }}
        quay_username: ${{ secrets.QUAY_USERNAME }}
        quay_password: ${{ secrets.QUAY_PASSWORD }}
        github_token: ${{ secrets.GITHUB_TOKEN }}
```
