# Retrieve-And-Rank-Lab
This is a fixed version of the lab originally hosted [here](https://www.ibm.com/watson/developercloud/doc/retrieve-rank/tutorial.shtml).

## Provision service instance Retrieve and Rank on Bluemix and create service credentials


## Set service credentials in Git Bash

```
RETRIEVE_AND_RANK_USER=<userIdFromServiceInstance>
RETRIEVE_AND_RANK_PASSWORD=<passwordFromServiceInstance>
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
  * example in Git Bash on Windows: `DOWNLOADS_FOLDER=/C/Users/IBM_ADMIN/Downloads`

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

* Try running the following command in Git Bash:
```
python ./train.py -u "${RETRIEVE_AND_RANK_USER}":"${RETRIEVE_AND_RANK_PASSWORD}" -i ${DOWNLOADS_FOLDER}/cranfield-gt.csv -c ${SOLR_CLUSTER_ID} -x example_collection -n "example_ranker"
```

You will see a failure. The reason is that the python script was not designed with Windows architecture in mind (it is not a platform-independent script).
It assumes certain things about the OS architecture which is true on Linux and Mac OS machines but not windows. But fear not!
The Bluemix Continuous Delivery Pipeline in Toolchains has you covered. Every job of every stage in the pipeline runs in a Linux "container" So you can have it run the python
script for you.



##### Mac OS

* Simply run the following in a terminal:

```
python ./train.py -u "${RETRIEVE_AND_RANK_USER}":"${RETRIEVE_AND_RANK_PASSWORD}" -i ${DOWNLOADS_FOLDER}/cranfield-gt.csv -c ${SOLR_CLUSTER_ID} -x example_collection -n "example_ranker"
```




#### Run python script in a pipline tool within a toolchain

Next we will try running the python script in a Delivery Pipline in Bluemix. We
will in fact have you go through the steps of manually creating your own empty
toolchain, adding the pipeline tool to it, adding the shell script to the pipeline
and kicking it off. We could've automated this entire process for you with a
`Create Toolchain` button but then you wouldn't appreciate all the work being
done for you behind the scenes!


##### Create your empty toolchain

* First go to [bluemix.net/devops](https://console.ng.bluemix.net/devops/getting-started)
which should lead you to the Getting Started page
for `Continuous Delivery` in Bluemix.
* You'll want to click the `Create a toolchain` button here.

![create-toolchain-button-on-getting-started-image](./toolchain-images/toolchains-getting-started.png)

* On the next page, under the "Other Templates" section (we're all pros, aren't we?) select `Build your own toolchain`.

![build-your-own-toolchain-image](./toolchain-images/build-your-own-toolchain.png)

* Finally, ensure the `Organization` the toolchain is being created is your own personal one and then give your toolchain an appropriate name on the next page (we suggest `retrieve-and-rank-toolchain`) and then hit `Create`.

![retrieve-and-rank-toolchain-creation](./toolchain-images/retrieve-and-rank-toolchain.png)

##### Add GitHub and Delivery Pipeline tools to run the script

First off, you will need to own source code repository to store your scripts and other assets in. For this, Continuous Delivery supports GitHub with more source code targets coming soon.

###### Add GitHub tool

* Tools can be added from the Tool Catalog, to get there from your shiny new toolchain click the `Add a Tool` button on the top-right.

![add-tool-button](./toolchain-images/add-tool-button.png)

* Now that you're on the Tool Catalog page, select the GitHub tool (feel free to use the search box at the top to filter).

![github-tool-catalog](./toolchain-images/github-tool-catalog.png)

* In case this is the first time you are using Continuous Delivery with GitHub, you will most likely have to `Authorize` the Continuous Delivery service to have access to your GitHub account here. If that's the case, click the button. This access allows us to do things you ask us to such as forking, cloning or accessing repositories on GitHub on your behalf. In case you are worried about granting this access, you can use a brand new GitHub account to use with us.

* Once authorized, you should see the configuration page below. Here we'd like to select `Clone` in the first dropdown and type the name of your repository in the second input box, such as `Retrieve-And-Rank-Lab`. For the 3rd and final input box, specify the location of this repository to "fork":  `https://github.com/Bluemix-Watson-Labs/Retrieve-And-Rank-Lab`. You may also want to uncheck the `Enable GitHub issues` checkbox, as we won't be using it in this lab. Once completed, click the `Create Integration` button.

![configure-github](./toolchain-images/configure-github.png)

* Once you've added the GitHub tool, watch the fancy little animation while it gets created. Note that if you've already forked this project into your GitHub account in the past you may get an error while it is configuring. If that's the case just go into the errored tool and change the `Repository Type` to `Existing` rather than `Fork`, hit `Save` and it should work.

* Once that is all configured,  you should see a page as below. If you see it you are ready to move on to the next step, which is to add a `Delivery Pipeline` tool.

![configured-github](./toolchain-images/github-configured.png)

###### Add Delivery Pipeline tool

* Once again, click the `Add a Tool` button on the top-right page of your toolchain.

* Search or scroll down to find the `Delivery Pipeline` tool and click on it.

![pipeline-tool-catalog](./toolchain-images/pipeline-tool-catalog.png)

* In the configuration page for it, optionally give it a name like `train-ranker-pipeline` but leave the checkbox unchecked, as in this case we are just running a script, there is no user interface associated with the deployment. Once you've given it a name, hit `Create Integration`.

![pipeline-config](./toolchain-images/pipeline-config.png)

#### Configure your Delivery Pipeline to run the script

* Now that your toolchain has the necessary tools, click into the Pipeline tool card once it's `Configured`.

![pipeline-card-configured](./toolchain-images/pipeline-card-configured.png)

* Welcome to the magical Pipeline User Interface! It's quite barebones right now, but we'll fix that shortly. Hit the `Add Stage` button.

![pipeline-barebones-ui](./toolchain-images/pipeline-barebones-ui.png)

* The very first step of any pipeline you create is generally a build step to pull in any source code from wherever it is located to begin with. So firstly, give the new stage a name like `Pull scripts`. Under this initial `Input` tab, that's all you need to do, as the defaults for everything else should already be set based on the contents of your toolchain (which has a single Github repo). Move on to the `Jobs` tab.

![pipeline-build-input-tab](./toolchain-images/pipeline-build-input-tab.png)

* Every Pipeline stage is made up of 1 or more jobs. So click the `Add Job` button in the `JOBS` tab and select `Build` type from the drowpdown. The default `Simple` build is enough for our purposes (it will just package up the contents of your repository for subsequent stages where the actual stuff happens). Once that is all done, hit `Save` at the bottom to return to the main Pipeline page.

![pipeline-build-job](./toolchain-images/pipeline-build-job.png)

* Once you're back at the main pipeline page, you should see the stage you just created but that isn't enough yet as you haven't actually created a stage to run your train-ranker python script! Let's create a second stage by clicking the `Add Stage` button.

![pipeline-one-stage-down-one-to-go](./toolchain-images/pipeline-one-stage-down-one-to-go.png)

* Let's call this next stage `Train Ranker` and then move on to the `JOBS` tab as the default `INPUT` for this stage should be correct already.

![pipeline-train-ranker-input](./toolchain-images/pipeline-train-ranker-input.png)

* Under the `JOBS` tab, click the `ADD JOB` button and select `Deploy` this time around. For this `Deploy` job name, let's call it `Run python script`. For everything else, you can likely leave the defaults (ensure it is using your personal org, and the `dev` space perhaps). Once you are down to the `Deploy Script` section enter the following in it (remove the current contents, including the line that says `cf push "${CF_APP}"`).

```
#!/bin/bash
python --version
python ./train-ranker.py -u ${RANK_AND_RETRIEVE_USER}:${RANK_AND_RETRIEVE_PASSWORD} -i ./cranfield-gt.csv -c ${SOLR_CLUSTER_ID} -x example_collection -n "example_ranker"
```

![pipeline-run-python-script](./toolchain-images/pipeline-run-python-script.png)

* If you have a keen eye, you'll notice that we actually used environment variables in the above little scriptlet. Let's define those in the 3rd and final tab in this Pipeline stage, the aptly named `ENVIRONMENT PROPERTIES` tab.

![pipeline-env-vars](./toolchain-images/pipeline-env-vars.png)

* Use the `Add Property` button multiple times to add the following env vars that were used by the scriptlet we pasted and fill in the value as appropriate (hopefully you kept the values noted down somewhere)!
  * Text Property: `RANK_AND_RETRIEVE_USER`
  * Secure Property: `RANK_AND_RETRIEVE_PASSWORD`
  * Text Property: `SOLR_CLUSTER_ID`

![pipeline-env-vars-filled](./toolchain-images/pipeline-env-vars-filled.png)

* Once the configuration is completely filled in, hit that `Save` button to return to the main Pipeline page. Once there, let's kick off the pipeline to watch the fruit of our labour come to life by clicking the Play (`Run Stage`) button on the `Pull Scripts` stage - see image below.

![run-pipeline](./toolchain-images/run-pipeline.png)

* There we go! You've trained the ranker. You can view the logs for the stage running by clicking on the `View logs and history` link in the Train Ranker stage, do so now and record the `ranker_id` to be used in your future curl commands.

```
RANKER_ID=<yourRankerID>
```


#### Retrieve some answers

While you're waiting for the ranker to finish training, you can search your documents. This search, which uses the "Retrieve" part of the Retrieve and Rank service, does not use the machine learning ranking features. It's a standard Solr search query.

* Try the following retrieve curl while the ranker is training:

```
curl https://${RETRIEVE_AND_RANK_USER}:${RETRIEVE_AND_RANK_PASSWORD}@gateway.watsonplatform.net/retrieve-and-rank/api/v1/solr_clusters/${SOLR_CLUSTER_ID}/solr/example_collection/select?q=what%20is%20the%20basic%20mechanism%20of%20the%20transonic%20aileron%20buzz&wt=json&fl=id%2Ctitle
```
  * Note: The /select request handler in the cURL command indicates that this is a standard Solr search query, not a query that uses the Rank portion of the Retrieve and Rank service. Standard Solr search results are unordered and might not contain the most significant results. In Stage 6, we will use the /fcselect request handler with a trained ranker to obtain results that are ordered (ranked) by significance.

* Be sure to hit `CTRL+C` to cancel out of the request once the response is received.

* Your query returns the 10 most relevant results from Solr.

* If you want to try other queries, look in the cranfield-gt.csv that you downloaded above (a copy of it is also in this repo). Copy a question from the first column and paste it as the value of the q parameter in your browser. Note before you use it, you must also url-encode it. You can Google how to url encode it, if you have Chrome press `F12` to open the Developer tools, switch to the `Console` tab and type the function name `encodeURIComponent('can the three-dimensional problem of a transverse potential flow about a body of revolution be reduced to a two-dimensional problem.')` to encode the question and `encodeURIComponent('id,title')` to encode that query param. For example you may get:

```
&q=can%20the%20three-dimensional%20problem%20of%20a%20transverse%20potential%20flow%20about%20a%20body%20of%20revolution%20be%20reduced%20to%20a%20two-dimensional%20problem.&wt=json&fl=id%2Ctitle
```

#### Re-rank the results

* Check the status of the ranker until you see a status of Available. With this sample data, training takes about 5 minutes and started when the pipeline finished earlier. Run the following command to test the ranker status.

```
curl -u "${RETRIEVE_AND_RANK_USER}":"${RETRIEVE_AND_RANK_PASSWORD}" "https://gateway.watsonplatform.net/retrieve-and-rank/api/v1/rankers/${RANKER_ID}"
```

* Once trained, query the ranker to view reranked results. See the example curl below, note that the `select` parameter has been changed to `fcselect`.

```
curl "https://${RETRIEVE_AND_RANK_USER}:${RETRIEVE_AND_RANK_PASSWORD}@gateway.watsonplatform.net/retrieve-and-rank/api/v1/solr_clusters/${SOLR_CLUSTER_ID}/solr/example_collection/fcselect?ranker_id=${RANKER_ID}&q=what%20is%20the%20basic%20mechanism%20of%20the%20transonic%20aileron%20buzz&wt=json&fl=id%2Ctitle"
```

* After evaluating the reranked search results, you can refine them by repeating the steps above starting with training the ranker. You can also add new documents, as described in the stage before training the ranker, to broaden the scope of the search. Repeat the process until you are completely satisfied with the results. This can require multiple iterations of refining and reranking.

* As above, you can experiment with other results. For example with:

```
&q=can%20the%20three-dimensional%20problem%20of%20a%20transverse%20potential%20flow%20about%20a%20body%20of%20revolution%20be%20reduced%20to%20a%20two-dimensional%20problem.&wt=json&fl=id%2Ctitle
```

#### Clean up

* You might want to delete the Solr components and ranker that you created in this tutorial. To clean up, you delete the cluster and ranker you created previously.

* Delete the cluster

```
curl -i -X DELETE -u "${RETRIEVE_AND_RANK_USER}":"${RETRIEVE_AND_RANK_PASSWORD}" "https://gateway.watsonplatform.net/retrieve-and-rank/api/v1/solr_clusters/${SOLR_CLUSTER_ID}"
```
* When the cluster is deleted, the response is HTTP response 200.

* Delete the ranker

```
curl -X DELETE -u "${RETRIEVE_AND_RANK_USER}":"${RETRIEVE_AND_RANK_PASSWORD}" "https://gateway.watsonplatform.net/retrieve-and-rank/api/v1/rankers/${RANKER_ID}"
```

* When the ranker is deleted, the response is an empty JSON object.

For more details as well as what you may want to do next see the original tutorial [here](https://www.ibm.com/watson/developercloud/doc/retrieve-rank/tutorial.shtml).
