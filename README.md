# Paebbl - Automation requirements to support Gwen

Points tackled:

## 1. Dockerfile generation and troubleshooting with python libraries

- 🟢 Dockerfile adjusted based on meeting with Gwen
- 🟢 Docker image successfully generated
- 🟢 Docker container up and running with Flask Application

## 2. How to automate every day work

### Source code updates to `main` branch should build new image

- 🟢 Github Actions pipeline should watch changes and trigger the process.

### Docker image should be pushed to image registry

(TBD) Image Registry options:

1. Docker Hub
2. Github Registry

https://docs.github.com/en/actions/use-cases-and-examples/publishing-packages/publishing-docker-images#publishing-images-to-docker-hub-and-github-packages

3. AWS ECR (Elastic Container Registry)

### Python App (Flask) within AWS should be re-deployed with new Docker image

- [Work in progress] Add Github actions steps to deploy Flask app to ECS.

### Deployment notifications

- 🟢 Github Actions step added for Slack notifications

- Slack Channel should be created for notifications.
- Data Scientists & Platform Team should have access to said channel.

<img src="https://github.com/juanroldan1989/terraform-url-shortener/raw/main/screenshots/slack-notification-from-pipeline.png" width="100%" />

## 3. Improvements to current workflow

Current state:

1. Software relies on CSV file contents to generate results.
2. CSV file `clean.csv` is included **within** Docker image for testing and development purposes.

Improvements:

1. Update python app to work with **external** CSV file.
   -> Python library to fetch data from Sharepoint:
   https://python.plainenglish.io/list-download-files-from-sharepoint-using-python-44cfc0793397

2. Current codebase will fetch latest data from Sharepoint.

3. Codebase will fallback to existing CSV file if Sharepoint API is not responsive.

4. Future:

- **S3 Bucket** provides drag-and-drop UI for colleagues to securely share files there.

- AWS SDK to be included within codebase for authentication puroses before downloading/uploading files.

# AWS ECR Repository setup

- `ECR_REGISTRY` -> `<aws_account_id>.dkr.ecr.<region>.amazonaws.com`
- `ECR_REPOSITORY` -> `paebbl-repository`
- `IMAGE_TAG` -> latest commit SHA from Github repository

1. Create AWS Docker Image registry

```ruby
$ aws ecr create-repository --repository-name $ECR_REPOSITORY --region <region>
```

2. Build Docker image:

```ruby
$ docker build -t flask-app .
```

3. Tag Docker image (matching new release):

```ruby
$ docker tag flask-app $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```

4. Push Docker image to registry:

```
$ aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin $ECR_REGISTRY

$ docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```

# AWS ECS Cluster setup
