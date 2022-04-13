# DataAnalyticsChallengeLab

The goal of this challenge lab is for you to analyze a set of e-commerce data to gain customer insight and use Artificial Intelligence to understand and anticipate risks & opportunities. Additionally, you will create a video object tracking pipeline prototype using Cloud AI and the Video Analysis API.

# Scenario
![image](https://user-images.githubusercontent.com/102045161/163252403-cd300f43-503f-4db8-9acd-55ab5f08e1ae.png)

You are an engineer at Cymbal Direct. Due to COVID-19 and shifts in customer preferences, they are looking for a way to more quickly understand customer patterns and better coordinate inventory between brick-and-mortar and web properties.

They want to use this data to make decisions about sourcing and promotions as well as orchestrating customer interactions. Some of the data they will be working with contains Personal Identifiable Information (PII) that needs to be protected.

Not only does Cymbal Direct want to understand customer behaviors, but they would also like to anticipate and respond to risks and opportunities using Artificial Intelligence.

In particular, they would like to identify anomalies in real-time transaction and clickstream data, as well as enhance the timeliness of the data going into recommendations. They would also like to deploy an AI or BQML model to production and manage the CI/CD process as new data comes in and their model improves.

You are tasked with helping Cymbal Direct accomplish these goals. You will do this over the course of two stages. In the first, you will deploy an analytics pipeline to help Cymbal Direct evaluate their transactions and e-commerce purchases in real time.

In the second stage, you will build a streaming video object tracking pipeline that utilizes Cloud AI and the Video Analysis API. This is a prototype that Cymbal Direct will later customize to fit their specific needs.

# Part 1: Analytics pipeline

1) Configure your environment.
2) Submit a Cloud Build job to containerize a Javascript app that sends JSON data to Cloud PubSub.
3) Deploy the PubSub JSON container app using Cloud Run.
4) Create a sample BigQuery dataset and table for test ecommerce data.
5) Create a Pub/Sub topic and subscription.
6) Deploy the Dataflow "Cloud PubSub Subscription to BigQuery" template to consume PubSub events.
7) Post JSON data to your fully managed service to test the pipeline.
8) Write BigQuery queries to test the pipeline.

# Part 2: Video object tracking pipelines

1) Configure your environment.
2) Create a Pub/Sub notification for Cloud Storage.
3) Create a BigQuery dataset and table.
4) Creating Pub/Sub topics and subscriptions for filtered results and error catching.
5) Create a Dataflow flex template job.
6) Upload sample video files.
7) Analyze video annotations in BigQuery.
8) Consume filtered results.
