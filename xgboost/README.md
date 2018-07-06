## Ames housing value prediction using XGBoost on Kubeflow

In this example we will demonstrate how to use Kubeflow with XGBoost. We will do a detailed
walk-through of how to implement, train and serve the model. You will be able to run the exact same workload on-prem and/or on any cloud provider. We will be using [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) to show how the end-to-end workflow runs on the cloud. 

## Steps
 * [Kubeflow Setup](#kubeflow-setup)
 * Data Preparation
 * Model Training
 * Model Export
 * Model Serving

### Kubeflow Setup
In this part you will setup Kubeflow on an existing Kubernetes cluster. Checkout the Kubeflow [setup guide](https://github.com/kubeflow/kubeflow#setup). 

### Data Preparation
Download the dataset from the [Kaggle competition](https://www.kaggle.com/c/house-prices-advanced-regression-techniques/data). 

### Model Training

#### Dockerfile
We have attached a Dockerfile with this repo which you can use to create a
docker image. We have also uploaded the image to gcr.io, which you can use to
directly download the image.

```
IMAGE_NAME=ames-housing
VERSION=v1
```

Let's create a docker image from our Dockerfile

```
docker build -t ${IMAGE_NAME}:${VERSION} .
```

Once the above command is successful you should be able to see the docker
images on your local machine `docker images` and then upload the image to
Google Container Registry using

```
gcloud docker -- push gcr.io/${PROJECT_ID}/${IMAGE_NAME}:${VERSION}
```

You can play with the image locally by performing

```
docker run -v /tmp/ames/:/model/ames -it $IMAGE_ID --train-input examples/xgboost/ames_dataset/train.csv --model-file /model/ames/housing.dat --learning-rate 0.1 --n-estimators 30000 --early-stopping-rounds 50
```

### Model Export
Model export goes here

### Model Serving
Model serving goes here

### Pre-requisites

As a part of running this setup, make sure you have enabled the Google
Kubernetes Engine API. In addition to that you will need to install
[Docker](https://docs.docker.com/install/) and [gcloud](https://cloud.google.com/sdk/downloads).

### Dockerfile
We have attached a Dockerfile with this repo which you can use to create a
docker image. We have also uploaded the image to gcr.io, which you can use to
directly download the image.

```
IMAGE_NAME=ames-housing
VERSION=v1
```

Let's create a docker image from our Dockerfile

```
docker build -t ${IMAGE_NAME}:${VERSION} .
```

Once the above command is successful you should be able to see the docker
images on your local machine `docker images` and then upload the image to
Google Container Registry using

```
gcloud docker -- push gcr.io/${PROJECT_ID}/${IMAGE_NAME}:${VERSION}
```

You can play with the image locally by performing

```
docker run -i -t gcr.io/${PROJECT_ID}/${IMAGE_NAME}:${VERSION} /bin/bash
```

### GKE and XGBoost
In the rest of the setup we will show you how to create cluster on GKE and
perform XGBoost training. Run the command below to create a 3 node cluster,
which should take few minutes. Checkout the [GKE
hello-app](https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app)
to see the set of commands used below

```
CLUSTER_NAME=zillow-xgboost
NUM_NODES=3
ZONE=us-central1-b

gcloud container clusters create ${CLUSTER_NAME} --num-nodes=${NUM_NODES} --zone=${ZONE}
```

```
python housing.py --train-input housing_prices/train.csv --learning-rate 0.1 --n-estimators 30000 --model-file housing.dat --early-stopping-rounds 40
```

```
docker run -v $(pwd):/seldon_serve_xgboost seldonio/core-python-wrapper:0.7 /seldon_serve_xgboost HousingServe 0.1 seldonio
```

```
docker run -v /tmp/ames/:/examples2 -it gcr.io/cloudmlplat/ames:v1 --train-input examples/xgboost/ames_dataset/train.csv --model-file /examples2/housing.dat
```
