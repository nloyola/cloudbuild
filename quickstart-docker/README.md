https://cloud.google.com/cloud-build/docs/quickstart-build

## BUILD 
```
export LOCATION=us-central1
gcloud beta artifacts repositories create quickstart-docker-repo --repository-format=docker \
    --location=${LOCATION} --description="Docker repository"
gcloud artifacts repositories list
gcloud config get-value project
gcloud builds submit --tag ${LOCATION}-docker.pkg.dev/tennisbana1/quickstart-docker-repo/quickstart-image
gcloud builds submit --config cloudbuild.yaml

```
