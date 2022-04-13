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
##### Make a note of the Cloud run Service url. You will need it in later steps 

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
#### Task 6: Deploy a Dataflow template to process Cloud PubSub messages <br>

Deploy a Dataflow template to process Cloud PubSub messages into BigQuery <br>

```
export STAGING_TABLE=cepfpart1staging
gsutil mb -c standard -l ${REGION} gs://${VIDEO_CLIPS_BUCKET}
gcloud dataflow jobs run ecommerce-events-ps-to-bq-stream --gcs-location gs://dataflow-templates-us-central1/latest/PubSub_Subscription_to_BigQuery --region us-central1 --staging-location gs://${STAGING_TABLE}/temp --parameters inputSubscription=projects/${PROJECT_ID}/subscriptions/${ECOMMERCE_SUBSCRIPTION},outputTableSpec=$PROJECT_ID:retail_dataset.ecommerce_events
```
#### Task 7: Post JSON data to your fully managed service <br>

Test the ecommerce pipeline by submitting JSON object data to the Cloud Run service endpoint. <br>

The cloud run service url you captured in deploy step

```
curl -vX POST [CLOUD RUN SERVICE URL]/json -d @[JSON DATA FILE] --header "Content-Type: application/json"
```
Generate ecommerce view data using the ecommerce_view_event.json data file <br>
Generate ecommerce add_item data using the ecommerce_add_to_cart_event.json data file <br>
Generate ecommerce purchase data using the ecommerce_purchase_event.json data file <br>

#### Task 8: Query ecommerce event data from BigQuery <br>

##### Query pulls all event_datetime, event, user_id data from ecommerce_events table.

```
SELECT event_datetime, event, user_id  
FROM `qwiklabs-gcp-01-38493cf3fb53.retail_dataset.ecommerce_events`
```
##### Query pulls all event, transactions and revenue data from ecommerce_events table.

```
select 
    event, 
    count(distinct ecommerce.purchase.transaction_id) as transactions,
    sum(ecommerce.purchase.value) as revenue
from `qwiklabs-gcp-02-f1163f3067e7.retail_dataset.ecommerce_events`
group by event
having transactions > 0

```

### Cleanup <br>
Once you have verified that you have completed all of the above tasks by scoring the maximum number of points possible on each of the activity tracking components, open the navigation menu and select Dataflow.

Select your ecommerce Dataflow job and click Stop > Cancel. Move on to the following section once you've verified that your ecommerce job has fully halted.





