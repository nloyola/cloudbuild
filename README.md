## cloudbuild
```
https://kevinsimper.medium.com/building-docker-images-with-docker-compose-on-google-cloud-container-builder-292b1eb3fd31
https://cloud.google.com/cloud-build/docs/deploying-builds/deploy-cloud-run
```

### Check local dev environment 
```
source env/bin/activate
cd backend
python manage.py runserver 0.0.0.0:8000
```

### Create a project with billing on google cloud
```
Make sure you have an account with $300 available and enable billing
You will enter credit card but they claim they wont auto bill after 3 months.
Enable ''ContainerRegistry'' for the project.
Create json key file and download
cat keyfile.json | docker login -u _json_key --password-stdin https://gcr.io
gcloud auth login
gcloud run deploy cloudservice1 --project ${PROJECT} --image gcr.io/${PROJECT}/cloudbuild1 --platform managed
```

### run container with runserver 
```
docker login
export PROJECT_ID=tennisbanan1
docker build -f Dockerfile.runserver . --tag gcr.io/${PROJECT_ID}/cloudbuild1
docker push gcr.io/${PROJECT_ID}/cloudbuild1
docker run -p 8080:8080 -e PORT=8080   gcr.io/${PROJECT_ID}/cloudbuild1
```

### Run deploy to cloud service
```
gcloud run deploy cloudservice1 --project ${PROJECT_ID} --image gcr.io/${PROJECT_ID}/cloudbuild1 --platform managed
```

### use docker-compose to run locally
```

docker build . --tag gcr.io/${PROJECT_ID}/cloudbuild2
# The following command may require 
docker push gcr.io/${PROJECT_ID}/cloudbuild2
docker-compose up
```

### with docker-compose cloudrun is no longer an option


### wipe docker clean of images and containers.
```
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
docker rmi $(docker images -q)
```

https://cloud.google.com/cloud-build/docs/quickstart-build

## BUILD 
```
export LOCATION=europe-north1
export PROJECT_ID=$(gcloud config list --format='value(core.project)')
gcloud beta artifacts repositories create quickstart-docker-repo --project=${PROJECT_ID} --repository-format=docker \
    --location=${LOCATION} --description="Docker repository"
gcloud artifacts repositories list
gcloud config get-value project

# The following is not necessary, probably costs money and fails
# gcloud builds submit --tag ${LOCATION}.pkg.dev/${PROJECT_ID}/quickstart-docker-repo/quickstart-image
```

## DEPLOY
```
export LOCATION=europe-north1
export PROJECT_ID=$(gcloud config list --format='value(core.project)')
export PROJECT_ID_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:$PROJECT_ID_NUMBER@cloudbuild.gserviceaccount.com \
    --role=roles/run.admin
gcloud iam service-accounts add-iam-policy-binding \
    $PROJECT_ID_NUMBER-compute@developer.gserviceaccount.com \
    --member=serviceAccount:$PROJECT_ID_NUMBER@cloudbuild.gserviceaccount.com \
    --role=roles/iam.serviceAccountUser
#Edit the yaml file and deploy the yaml with the following command
gcloud builds submit --config cloudbuild.yaml
```
