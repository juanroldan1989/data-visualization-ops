# Paebbl - Automation requirements to support Gwen

Points tackled:

## 1. Dockerfile generation and troubleshooting with python libraries

🟢 Dockerfile adjusted based on meeting with Gwen:

```ruby
# syntax=docker/dockerfile:1.4
FROM --platform=$BUILDPLATFORM python:3.10-alpine AS builder

WORKDIR /app

COPY requirements.txt /app
RUN --mount=type=cache,target=/root/.cache/pip \
  pip3 install -r requirements.txt

COPY . /app

ENTRYPOINT ["python3"]
CMD ["app.py"]
```

## 2. How to automate every day work

### Source code updates to `main` branch should build new image

- 🟢 Github Actions pipeline should watch changes and trigger the process.

### Python App (Flask) within AWS should be re-deployed with new Docker image

- [Work in progress] Add Github actions steps to deploy Flask app to ECS.

### Notify Platform Team about deployments

- 🟢 Github Actions step added for Slack notifications to notify Data Scientists & Platform Team.

## 3. Improvements to Gwen's workflow

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
