# Paebbl - Data Visualization Ops

1. [Primary Goal](#primary-goal)
2. [Milestones](#milestones)
3. [Secure App Access](#secure-app-access)

[Slides](https://docs.google.com/presentation/d/160hWqh9vSh-fF2EI2x-cUtoE3KtobIK2BL_jAX808lU/edit?usp=sharing)

# Primary Goal

Make data visualisation work accessible to colleagues

# Milestones

## 1. Dockerise App

Dockerfile generation and troubleshooting with python libraries.

- 🟢 Dockerfile adjusted based on meeting with Gwen
- 🟢 Docker image successfully generated
- 🟢 Docker container up and running with Flask Application

## 2. Pipeline

How to automate every day work:

- 🟢 Source code updates to `main` branch should build new image
- 🟢 Github Actions pipeline should watch changes and trigger the process.
- 🟢 Docker image should be pushed to image registry

https://docs.github.com/en/actions/use-cases-and-examples/publishing-packages/publishing-docker-images#publishing-images-to-docker-hub-and-github-packages

## 3. Deployment

Python App (Flask) within AWS should be re-deployed with new Docker image:

- 🟢 Add Github actions steps to deploy Flask app to ECS.

### Deployment notifications

- 🟢 Github Actions step added for Slack notifications

- Slack Channel should be created for notifications.
- Data Scientists & Platform Team should have access to said channel.

<img src="https://github.com/juanroldan1989/terraform-url-shortener/raw/main/screenshots/slack-notification-from-pipeline.png" width="100%" />

# Secure App Access

https://aws.amazon.com/blogs/containers/securing-amazon-elastic-container-service-applications-using-application-load-balancer-and-amazon-cognito/

![Cognito-ALB-Authentication-with-Fargate-Title](https://github.com/user-attachments/assets/2968b9b4-f1ce-4c79-9a64-6b0be3cd0208)

![hostedUI-login-IdP-selection](https://github.com/user-attachments/assets/1a537a40-993d-4140-aae3-0db4c96dd1ae)

- Authenticate users accessing containerized application **without writing authentication code**.

- Using ALB inbuilt integration with Amazon Cognito.

- Authentication **is offloaded** from the application.

## Improvements

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
- `REGION` -> `us-east-1`
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
$ aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $ECR_REGISTRY

Login Succeeded

$ docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```

# AWS ECS Cluster setup

1. Create ECS Cluster

```ruby
ecs-cli configure \
  --region us-east-1 \
  --cluster ecs-paebbl-cluster \
  --default-launch-type EC2 \
  --config-name ecs-paebbl-cluster

aws ecs create-cluster \
  --cluster-name ecs-paebbl-cluster \
  --service-connect-defaults namespace=paebbl-namespace \
  --region us-east-1 \
  --tags key=app,value=ecs-paebbl-cluster key=owner,value=paebbl \
  --settings '{ "name": "containerInsights", "value": "enabled" }'
```

2. Set Capacity Providers:

```ruby
aws ecs put-cluster-capacity-providers \
  --cluster ecs-paebbl-cluster \
  --capacity-providers EC2 \
  --default-capacity-provider-strategy capacityProvider=EC2,weight=1 \
  --region us-east-1
```

3. Register ECS Task that representes application:

```ruby
aws ecs register-task-definition \
  --region us-east-1 \
  --requires-compatibilities EC2 \
  --cli-input-json file://flask/app/task-definition-app.json
```

4. Create ECS Service that serves application to users:

```ruby
aws ecs run-task \
  --cluster ecs-paebbl-cluster \
  --task-definition task-definition-app:5 \
  --count 1 --launch-type EC2
```

```ruby
aws ecs create-service \
  --cluster ecs-paebbl-cluster \
  --service-name "app" \
  --desired-count 1 \
  --task-definition "task-definition-app" \
  --launch-type EC2 \
  --service-connect-configuration '{
    "enabled": true,
    "namespace": "paebbl-namespace",
    "services":
    [
        {
          "portName": "app-port",
          "clientAliases": [
            {
                "port": 80,
                "dnsName": "app"
              }
            ]
        }
      ]
  }'
```
