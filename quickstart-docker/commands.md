https://cloud.google.com/cloud-build/docs/quickstart-build

## BUILD 
```
gcloud beta artifacts repositories create quickstart-docker-repo --repository-format=docker \
    --location=us-central1 --description="Docker repository"
gcloud artifacts repositories list
gcloud config get-value project
gcloud builds submit --tag us-central1-docker.pkg.dev/tennisbana1/quickstart-docker-repo/quickstart-image:tag1
```
