## Challenge scenario
As a junior data engineer in Jooli Inc. and recently trained with Google Cloud and a number of data services you have been asked to demonstrate your newly learned skills. The team has asked you to complete the following tasks.


### Task 1: Run a simple Dataflow job

- Open Cloud Shell

```
gsutil cp gs://cloud-training/gsp323/lab.schema .

cat lab.schema

```

- Cloud Storgage => Create Bucket with [PROJECT_ID]

- BigQuery => create dataset 'lab' from [PROJECT_ID]

- Create table from Dataset 'lab'

* table from gs://cloud-training/gsp323/lab.csv

* Table name : customers
* Schema : enable "Edit As Text" 
```
 [
        {"type":"STRING","name":"guid"},
        {"type":"BOOLEAN","name":"isActive"},
        {"type":"STRING","name":"firstname"},
        {"type":"STRING","name":"surname"},
        {"type":"STRING","name":"company"},
        {"type":"STRING","name":"email"},
        {"type":"STRING","name":"phone"},
        {"type":"STRING","name":"address"},
        {"type":"STRING","name":"about"},
        {"type":"TIMESTAMP","name":"registered"},
        {"type":"FLOAT","name":"latitude"},
        {"type":"FLOAT","name":"longitude"}
    ]
```


- DataFlow => Create Job From Template

* job name :  lab-transform

* DataFlow template :  Dataflow batch template => Text Files on Cloud Storage to BigQuery

* JavaScript UDF path in Cloud Storage
gs://cloud-training/gsp323/lab.js
* JSON path
gs://cloud-training/gsp323/lab.schema
* JavaScript UDF name
transform
* BigQuery output table
YOUR_PROJECT:lab.customers
* Cloud Storage input path
gs://cloud-training/gsp323/lab.csv
* Temporary BigQuery directory
gs://YOUR_PROJECT/bigquery_temp
* Temporary location
gs://YOUR_PROJECT/temp

* Run 

### Task 2: Run a simple Dataproc job

- Dataproc => Create Cluster with default values

- When new Cluster is ready, click on the cluster name => VM Instance => SSH

- Run 
 
```
 hdfs dfs -cp gs://cloud-training/gsp323/data.txt /data.txt
```

- Job => Sumbit a job 

* Region	us-central1
* Job type	Spark
* Main class or jar	org.apache.spark.examples.SparkPageRank
* Jar files	file:///usr/lib/spark/examples/jars/spark-examples.jar
* Arguments	/data.txt
* Max restarts per hour	1

- Run


### Task 3: Run a simple Dataprep job


- Open Dataprep

- Cloud Storage =>  edit Cloud Storage 

gs://cloud-training/gsp323/runs.csv


- Continue


* select column10 => Filters Rows => On column value => is not "FAILURE" => Keep matching rows

* select column9 => Filters Rows => on column value => contains /(^0$|^0\.0$)/ => Deleting matching rows

* rename columns to runid, userid, labid, lab_title, start, end, time, score, state

- Run 

### Task 4: AI

- Create API KEY 

* Open API SERVICES

* CREDENTIALS => Create credentials => API KEY 

* Copy API KEY

* Open Cloud Shell

```
export API_KEY=[API_KEY]
```


- Use the Cloud Natural Language API to analyze the sentence from text about Odin. The text you need to analyze is "Old Norse texts portray Odin as one-eyed and long-bearded, frequently wielding a spear named Gungnir and wearing a cloak and a broad hat." Once you have analyzed the text you can upload the resulting analysis to gs://YOUR_PROJECT-marking/task4-cnl.result.

```
gcloud iam service-accounts create my-natlang-sa \ --display-name "my natural language service account" 

gcloud iam service-accounts keys create ~/key.json \ --iam-account my-natlang-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com 

export GOOGLE_APPLICATION_CREDENTIALS="/home/$USER/key.json" 

gcloud auth activate-service-account my-natlang-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com --key-file=$GOOGLE_APPLICATION_CREDENTIALS 

gcloud ml language analyze-entities --content="Old Norse texts portray Odin as one-eyed and long-bearded, frequently wielding a spear named Gungnir and wearing a cloak and a broad hat." > result.json 

gcloud auth login (Copy the token from the link provided) 

gsutil cp result.json gs://$GOOGLE_CLOUD_PROJECT-marking/task4-cnl.result


```



- Use Google Cloud Speech API to analyze the audio file gs://cloud-training/gsp323/task4.flac. Once you have analyzed the file you can upload the resulting analysis to gs://YOUR_PROJECT-marking/task4-gcs.result.

* vi request.json

```
{ "config": { "encoding":"FLAC", "languageCode": "en-US" }, "audio": { "uri":"gs://cloud-training/gsp323/task4.flac" } } 

```

```
curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json \ "https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" > result.json 

gsutil cp result.json gs://YOUR_PROJECT-marking/task4-gcs.result 

```

- Use Google Video Intelligence and detect all text on the video gs://spls/gsp154/video/train.mp4. Once you have completed the processing of the video, pipe the output into a file and upload to gs://YOUR_PROJECT-marking/task4-gvi.result. Ensure the progress of the operation is complete and the service account you're uploading the output with has the Storage Object Admin role.

```

gcloud iam service-accounts create quickstart 

gcloud iam service-accounts keys create key.json --iam-account quickstart@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com 


gcloud auth activate-service-account --key-file key.json 

export ACCESS_TOKEN=$(gcloud auth print-access-token) 
```



* vi request.json  

```

{ "inputUri":"gs://spls/gsp154/video/chicago.mp4", "features": [ "TEXT_DETECTION" ] } 
```

```
curl -s -H 'Content-Type: application/json' \ -H "Authorization: Bearer $ACCESS_TOKEN" \ ' https://videointelligence.googleapis.com/v1/videos:annotate' \ -d @request.json 

```

```
curl -s -H 'Content-Type: application/json' -H "Authorization: Bearer $ACCESS_TOKEN" ' https://videointelligence.googleapis.com/v1/operations/Informations_From_Previous_Request' > result1.json 
```

```
gsutil cp result1.json gs://YOUR_PROJECT-marking/task4-gvi.result 
```


















