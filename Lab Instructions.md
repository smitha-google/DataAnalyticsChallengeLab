## 0. Part 1 Analytics Pipeline

#### Task 1: Configure your environment <br>

Download tar file <br>

```
gsutil cp gs://sureskills-ql/challenge-labs/tech-bash-2021/data-analytics/data_analytics.tar.gzip .
tar -xvf data_analytics.tar.gzip
cd ~/data_analytics
```
#### Task 2: Submit a Cloud Build job <br>

Submit a Cloud Build Job to build the Javascript application to generate simulated ecommerce PubSub messages. <br>

##### 1. Create a Docker repository in Artifact Registry <br>

```
gcloud artifacts repositories create quickstart-docker-repo --repository-format=docker     --location=us-central1 --description="Docker repository"
gcloud artifacts repositories list
```
##### 2. Build an image using Dockerfile <br>

```
cd ~/data_analytics/pubsub_ecommerce
gcloud builds submit --tag us-central1-docker.pkg.dev/$PROJECT_ID/quickstart-docker-repo/pubsub-proxy:tag1
```
#### Task 3: Deploy a container using Cloud Run <br>

Deploy the containerized application to Cloud Run <br>

##### Get the image name from the prev build step - similar to this [us-central1-docker.pkg.dev/qwiklabs-gcp-01-982ea830b95e/quickstart-docker-repo/pubsub-proxy:tag1] <br>

##### Then use that image name to deploy
```
gcloud run deploy pubsub-proxy --image <image_name>
```
##### After you create the service - go to the Permissions page and add this - Only authenticated invocations are allowed for this service.
##### To allow unauthenticated invocations, add "allUsers" as a principal and assign it the "Cloud Run invoker" role.

#### Task 4: Create a BigQuery dataset and table <br>

Create a BigQuery dataset and table for simulated ecommerce event data. <br>

##### Create the BQ dataset via UI <br>
##### Create the Table with - 

```
bq mk --table $PROJECT_ID:retail_dataset.ecommerce_events bq_schema_ecommerce_events.json
```

#### Task 5: Create a Pub/Sub topic and subscription <br>

Create a Pubsub topic and subscription for simulated ecommerce event data <br>

```
export ECOMMERCE_TOPIC=ecommerce-events
export ECOMMERCE_SUBSCRIPTION=ecommerce-events-pull
gcloud pubsub topics create ${ECOMMERCE_TOPIC}
gcloud pubsub subscriptions create ${ECOMMERCE_SUBSCRIPTION} --topic=${ECOMMERCE_TOPIC}
```





