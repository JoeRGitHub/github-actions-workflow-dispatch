## GitHub Secrets

| Secret Name | Description |
| --- | --- |
| `DOCKERHUB_USER` | Your DockerHub username |
| `DOCKERHUB_TOKEN` | DockerHub password or PAT |

---

## `workflow_dispatch` Inputs

We will let the user choose:

- Which environment to deploy to: `staging` or `production`

And we’ll tag and push the image accordingly.

---

## `manual-push.yml` Workflow File

```yaml
name: Manual Docker Push

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Where to push the image?'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

env:
  DOCKERHUB_USER: ${{ secrets.DOCKERHUB_USER }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set DockerHub repo name
        id: set-repo
        run: |
          if [[ "${{ github.event.inputs.environment }}" == "production" ]]; then
            echo "repo=${DOCKERHUB_USER}/myapp-prod" >> $GITHUB_OUTPUT
          else
            echo "repo=${DOCKERHUB_USER}/myapp-staging" >> $GITHUB_OUTPUT
          fi

      - name: Log in to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and Push Docker Image
        run: |
          IMAGE=${{ steps.set-repo.outputs.repo }}
          docker build -t $IMAGE:latest .
          docker push $IMAGE:latest
```

---

- The user selects an `environment` input: `staging` or `production`
- We dynamically set the Docker image name (`myapp-staging` or `myapp-prod`)
- The image is built and pushed to the correct DockerHub repo

---

## Expected Outcome

After manually triggering the workflow:

- A Docker image is pushed to either:
    - `youruser/myapp-staging:latest`
    - or `youruser/myapp-prod:latest`

You can verify using:

```bash
docker pull youruser/myapp-staging:latest
```
