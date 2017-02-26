# Retrieve-And-Rank-Lab

## Provision service instance Retrieve and Rank on Bluemix and create service credentials


## Set service credentials in Git Bash

```
RETRIEVE_AND_RANK_USER=<userIdFromServiceInstance>
RETRIEVE_AND_RANK_PASSWORD==<passwordFromServiceInstance>
```

### Create a SOLR cluster

```
curl -X POST -u "$RETRIEVE_AND_RANK_USER":"$RETRIEVE_AND_RANK_PASSWORD" "https://gateway.watsonplatform.net/retrieve-and-rank/api/v1/solr_clusters" -d ""
```

Should respond with:
```
{"solr_cluster_id":"<cluster_id>","cluster_name":"","cluster_size":"","solr_cluster_status":"NOT_AVAILABLE"}
```

Store the solr_cluster_id and set as an environment variable like so:

```
SOLR_CLUSTER_ID=<solrClusterId>
```

### Create a collection and add documents

#### Wait until cluster is ready

```
curl -u "${RETRIEVE_AND_RANK_USER}":"${RETRIEVE_AND_RANK_PASSWORD}" "https://gateway.watsonplatform.net/retrieve-and-rank/api/v1/solr_clusters/${SOLR_CLUSTER_ID}"
```

Responds with something like this:

```
{"solr_cluster_id":"<cluster_id>","cluster_name":"","cluster_size":"","solr_cluster_status":"NOT_AVAILABLE"}
```

* Keep querying this API until the `solr_cluster_status` is `READY`.

#### Upload an example config

* Download this [zip file](https://github.com/watson-developer-cloud/doc-tutorial-downloads/raw/master/retrieve-and-rank/cranfield-solr-config.zip).

* Set the env var to your downloads folder like so:

```
DOWNLOADS_FOLDER=<pathToWhereYouDownloadFiles>
```

* Upload the config:

```
curl -X POST -H "Content-Type: application/zip" -u "${RETRIEVE_AND_RANK_USER}":"${RETRIEVE_AND_RANK_PASSWORD}" "https://gateway.watsonplatform.net/retrieve-and-rank/api/v1/solr_clusters/${SOLR_CLUSTER_ID}/config/example_config" --data-binary @${DOWNLOADS_FOLDER}/cranfield-solr-config.zip
```

#### Create an example collection

```
curl -X POST -u "${RETRIEVE_AND_RANK_USER}":"${RETRIEVE_AND_RANK_PASSWORD}" "https://gateway.watsonplatform.net/retrieve-and-rank/api/v1/solr_clusters/${SOLR_CLUSTER_ID}/solr/admin/collections" -d "action=CREATE&name=example_collection&collection.configName=example_config"
```

* After a few minutes it should respond with the following (if successful):

```
<?xml version="1.0" encoding="UTF-8"?>
<response>
<lst name="responseHeader"><int name="status">0</int><int name="QTime">12639</int></lst><lst name="success"><lst name="10.176.43.45:6113_solr"><lst name="responseHeader"><int name="status">0</int><int
 name="QTime">2870</int></lst><str name="core">example_collection_shard1_replica1</str></lst><lst name="10.176.43.136:6844_solr"><lst name="responseHeader"><int name="status">0</int><int name="QTime">
3220</int></lst><str name="core">example_collection_shard1_replica2</str></lst></lst>
</response>
```

#### Add the documents you will search
* Download the documents to search [here](
https://github.com/watson-developer-cloud/doc-tutorial-downloads/raw/master/retrieve-and-rank/cranfield-data.json).

```
curl -X POST -H "Content-Type: application/json" -u "${RETRIEVE_AND_RANK_USER}":"${RETRIEVE_AND_RANK_PASSWORD}" "https://gateway.watsonplatform.net/retrieve-and-rank/api/v1/solr_clusters/${SOLR_CLUSTER_ID}/solr/example_collection/update" --data-binary @${DOWNLOADS_FOLDER}/cranfield-data.json
```

* A successful response should look like the following (try a few times if it doesn't work the first time around).

```
{"responseHeader":{"status":0,"QTime":1785}}
```

#### Run the training Python Script

##### Windows

The python script was not designed with Windows in mind. It assumes certain things about
the OS architecture which is true on Linux and Mac OS machines but not windows. But fear not!
The Bluemix Continuous Delivery Pipeline in Toolchains has you covered. Every job of every
stage in the pipeline runs in a Linux "container" So you can have it run the python
script for you.

##### Mac OS

* Simply run the following in a terminal:

```
python ./train.py -u "${RETRIEVE_AND_RANK_USER}":"${RETRIEVE_AND_RANK_PASSWORD}" -i ${DOWNLOADS_FOLDER}/cranfield-gt.csv -c ${SOLR_CLUSTER_ID} -x example_collection -n "example_ranker"
```




#### Python script Toolchain
#### bluemix.net/devops
#### Toolchain tab
#### Add toolchain button
#### Empty toolchain (bottom)
#### Add a tool
