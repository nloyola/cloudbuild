## Cloudbuild Links
* [Building docker images with docker-compose on Google Cloud Container Builder](https://kevinsimper.medium.com/building-docker-images-with-docker-compose-on-google-cloud-container-builder-292b1eb3fd31)

* [Deploying to Cloud Run](https://cloud.google.com/cloud-build/docs/deploying-builds/deploy-cloud-run)

* [Quickstart: Getting started with Cloud SDK](https://cloud.google.com/sdk/docs/quickstart)

### Create a project with billing on google cloud
- Make sure you have an account with $300 available and enable billing
- You will enter credit card but they claim they wont auto bill after 3 months.
- Enable ''ContainerRegistry'' for the project.
- Create json key file and download
```sh
cat keyfile.json | docker login -u _json_key --password-stdin https://gcr.io
gcloud auth login
```

### Determine current Cloudbuild project

For project name (PROJECT_ID):

```sh
gcloud config list --format='value(core.project)'
```

For project number (PROJECT_ID_NUMBER):

```sh
gcloud config list --format='value(core.project)'
```

The values returned by these commands should be placed in the `.env` file below.

### Setup environment
```sh
cp .env.template .env

# modify file .env to suit your project.
source .env
```

### Check local dev environment
```sh
source env/bin/activate
cd backend
python manage.py runserver 0.0.0.0:8000
```

### run container locally  with runserver
```sh
source .env # optional
docker login
docker build -f Dockerfile.runserver . --tag gcr.io/${PROJECT_ID}/cloudbuild1
docker push gcr.io/${PROJECT_ID}/cloudbuild1
docker run -p 8080:8080 -e PORT=8080   gcr.io/${PROJECT_ID}/cloudbuild1
```

### deploy and run in  cloud service
```sh
gcloud run deploy cloudservice1 --project ${PROJECT_ID} --image gcr.io/${PROJECT_ID}/cloudbuild1 --platform managed
```

### use docker-compose to run locally
```sh
# I don't undersand why the Dockerfile needs runserver command

docker build . --tag gcr.io/${PROJECT_ID}/cloudbuild2
# The following command may require
docker push gcr.io/${PROJECT_ID}/cloudbuild2
docker-compose up
```


### wipe docker clean of images and containers.
```sh
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
docker rmi $(docker images -q) --force
```

## BUILD

* Review this link: [Quickstart: Build](https://cloud.google.com/cloud-build/docs/quickstart-build)

*Note there two images, `cloudbould1`  and `cloudbuild2`, identical except for:*

* `cloudbuild1` starts runserver as part of Dockerfile and is deployed by Dockerfile
* `cloudbuild2` only defines the image, and deploy is started by docker-compose

*Thus `cloudbuild2` cannot easily ( if at all) be deployed by cloudrun.*

```sh
source .env # optional
gcloud beta artifacts repositories create quickstart-docker-repo --project=${PROJECT_ID} --repository-format=docker \
    --location=${LOCATION} --description="Docker repository"
gcloud artifacts repositories list
gcloud config get-value project

```

## Deploy to cloud with Dockerfile defined container
```sh
source .env # optional
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:$PROJECT_ID_NUMBER@cloudbuild.gserviceaccount.com \
    --role=roles/run.admin
gcloud iam service-accounts add-iam-policy-binding \
    $PROJECT_ID_NUMBER-compute@developer.gserviceaccount.com \
    --member=serviceAccount:$PROJECT_ID_NUMBER@cloudbuild.gserviceaccount.com \
    --role=roles/iam.serviceAccountUser
# Edit the yaml file and deploy the yaml with the following command
# This cloudbuild takes the cloudimage1 container which does not use docker-compose and makes it available on the web
gcloud builds submit --config cloudbuild1.yaml
```
## Deploy to cloud with docker-compose defined image
###  Build locally with docker-compose build and deploy image
```sh
docker-compose build web
docker tag cloudbuild_web gcr.io/${PROJECT_ID}/cloudbuild_web
docker push gcr.io/${PROJECT_ID}/cloudbuild_web
gcloud run deploy cloudservice3  --project ${PROJECT_ID} --image gcr.io/${PROJECT_ID}/cloudbuild_web --platform managed
```

### run docker-compose created container via cloudbuild.yaml
```
gcloud builds submit --config cloudbuild4.yaml
```
