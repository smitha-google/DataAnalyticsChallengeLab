## Part 1 Video object tracking pipelines 

#### Task 1: Configure your environment <br>

```
gcloud services enable dataflow.googleapis.com containerregistry.googleapis.com videointelligence.googleapis.com
git clone https://github.com/GoogleCloudPlatform/dataflow-video-analytics.git
cd dataflow-video-analytics
```
#### Task 2: Create a Pub/Sub notification for Cloud Storage <br>

Create Cloud PubSub topics and subscriptions for storage event notifications. <br>

```
export PROJECT=[PROJECT]
export REGION=[REGION]
export GCS_NOTIFICATION_TOPIC="gcs-notification-topic"
export GCS_NOTIFICATION_SUBSCRIPTION="gcs-notification-subscription"

gcloud pubsub topics create ${GCS_NOTIFICATION_TOPIC}
gcloud pubsub subscriptions create ${GCS_NOTIFICATION_SUBSCRIPTION} --topic=${GCS_NOTIFICATION_TOPIC}
```
Create a Google Cloud Storage Bucket <br>

```
export VIDEO_CLIPS_BUCKET=${PROJECT}_videos
export DATAFLOW_TEMPLATE_BUCKET=${PROJECT}_dataflow_template_config
gsutil mb -c standard -l ${REGION} gs://${VIDEO_CLIPS_BUCKET}
gsutil notification create -t gcs-notification-topic -f json gs://${VIDEO_CLIPS_BUCKET}
gsutil mb -c standard -l ${REGION} gs://${DATAFLOW_TEMPLATE_BUCKET}
```

#### Task 3: Create a BigQuery dataset and table <br>

Create a BigQuery dataset and table for video object tracking analysis.

```
export BIGQUERY_DATASET="video_analytics"
bq mk -d --location=US ${BIGQUERY_DATASET}

bq mk -t \
--schema src/main/resources/table_schema.json \
--description "object_tracking_data" \
${PROJECT}:${BIGQUERY_DATASET}.object_tracking_analysis
```

#### Task 4: Creating Pub/Sub topics and subscriptions for filtered results and error catching

Create Cloud PubSub topic and subscription for object detection events. <br>
Create a Cloud PubSub topic and subscription for object detection error events. <br>

```
export OBJECT_DETECTION_TOPIC="object-detection-topic"
export OBJECT_DETECTION_SUBSCRIPTION="object-detection-subscription"
export ERROR_TOPIC="error-topic"
export ERROR_SUBSCRIPTION="error-subscription"
gcloud pubsub topics create ${OBJECT_DETECTION_TOPIC}
gcloud pubsub subscriptions create ${OBJECT_DETECTION_SUBSCRIPTION} --topic=${OBJECT_DETECTION_TOPIC}
gcloud pubsub topics create ${ERROR_TOPIC}
gcloud pubsub subscriptions create ${ERROR_SUBSCRIPTION} --topic=${ERROR_TOPIC}
```

#### Task 5: Create a Dataflow flex template job <br>

Use Gradle to build a flex template using the provided source code and deploy the flex template to Dataflow.

##### Gradle Build <br>
```
gradle spotlessApply -DmainClass=com.google.solutions.df.video.analytics.VideoAnalyticsPipeline
gradle build -DmainClass=com.google.solutions.df.video.analytics.VideoAnalyticsPipeline
```
##### Trigger using Gradle Run <br>
```
gradle run -Pargs="
--project=$PROJECT --region=us-central1
--runner=DataflowRunner --streaming --enableStreamingEngine
--autoscalingAlgorithm=THROUGHPUT_BASED --numWorkers=3 --maxNumWorkers=5 --workerMachineType=n1-highmem-4
--inputNotificationSubscription=projects/$PROJECT/subscriptions/$GCS_NOTIFICATION_SUBSCRIPTION
--outputTopic=projects/$PROJECT/topics/$OBJECT_DETECTION_TOPIC
--errorTopic=projects/$PROJECT/topics/$ERROR_TOPIC
--features=OBJECT_TRACKING --entities=cat --confidenceThreshold=0.9 --windowInterval=1 
--tableReference=$PROJECT:video_analytics.object_tracking_analysis"
```
##### Create a docker image for flex template <br>
```
gradle jib -Djib.to.image=gcr.io/${PROJECT}/dataflow-video-analytics:latest
```
##### Upload the template JSON config file to GCS.<br>

```
cat << EOF | gsutil cp - gs://${DATAFLOW_TEMPLATE_BUCKET}/dynamic_template_video_analytics.json
{
  "image": "gcr.io/${PROJECT}/dataflow-video-analytics:latest",
  "sdk_info": {"language": "JAVA"}
}
EOF
```
##### Trigger using Dataflow flex template <br>

```
gcloud beta dataflow flex-template run "video-object-tracking" \
--project=${PROJECT} \
--region=${REGION} \
--template-file-gcs-location=gs://${DATAFLOW_TEMPLATE_BUCKET}/dynamic_template_video_analytics.json \
--parameters=<<'EOF'
^~^autoscalingAlgorithm="NONE"~numWorkers=5~maxNumWorkers=5~workerMachineType=n1-highmem-4
  ~inputNotificationSubscription=projects/${PROJECT}/subscriptions/${GCS_NOTIFICATION_SUBSCRIPTION}
  ~outputTopic=projects/${PROJECT}/topics/${OBJECT_DETECTION_TOPIC}
  ~errorTopic=projects/${PROJECT}/topics/${ERROR_TOPIC}
  ~features=OBJECT_TRACKING~entities=window,person~confidenceThreshold=0.9~windowInterval=1
  ~tableReference=${PROJECT}:${BIGQUERY_DATASET}.object_tracking_analysis
  ~streaming=true
EOF
```
#### Task 6: Upload sample video files <br>

Upload the 5 second video file clips to your object detection Cloud Storage bucket.

##### Copy the files 
```
gsutil -m cp "gs://df-video-analytics-drone-dataset/*" .
```

##### Install ffmeg

```
Sudo apt install ffmeg
```
##### Split the files to 5 sec chunks for every single video file
```
myfile=VerticalFlyOver.mp4  
ffmpeg -i "$myfile" -codec:a aac  -ac 2  -ar 48k -c copy -movflags faststart -f segment -segment_format mpegts  -segment_time 5 "${myfile%.*}~"%1d.mp4
```
##### Copy the split file to the video bucket

```
gsutil -m cp ./*.mp4 gs://${PROJECT}_videos
```

#### Task 7: Analyze video annotations in BigQuery <br>

##### Query pulls all entity, file_name data from the object_tracking_analysis table

```
SELECT entity, file_name
FROM ${PROJECT}:${BIGQUERY_DATASET}.object_tracking_analysis
```

##### Query pulls all entity, max_confidence data from the object_tracking_analysis table

```
```

##### Query pulls all entity, processing_timestamp, timeOffset, confidence data from the object_tracking_analysis table

```
```


