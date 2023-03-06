# etl-tracking

This is a project that processes tracking data from different types of devices.
The data is ingested, transformed, and combined to form a dataset from which a user can view all the data as a whole.

## Features
The etl-tracking project currently contains the following:

* High-level architect overview
* NiFi xml pipeline template
* Docker compose file and accompanying environment file to deploy all necessary services

## Installation steps
1. Download the repository on your local machine
2. Open an integrated terminal in the root *etl-tracking-project* directory
3. Configure the *minio* service, in the **docker-compose.yaml** file, with a suitable `MINIO_ROOT_USER` username and `MINIO_ROOT_PASSWORD` password
4. Reflect the above configured username and password in the *createbuckets* service
5. To start the docker services, run `docker-compose up -d`
6. NiFi auto-generated credentials and can be accessed in the logs by running this command: `docker logs -f nifi`
```
eg
2023-03-03 15:20:27,167 INFO [main] o.a.n.a.s.u.SingleUserLoginIdentityProvider 

Generated Username [49608595-aca6-44ce-8458-874d01bfc734]
Generated Password [04xBLiRoS6LAi3ok5lt9sJO4KwRUyTyq]
```
8. Open NiFi at [Nifi](https://localhost:8443/nifi/)
9. Use the credentials to log in

**You can overwrite the generated credentials by running the following commands:**
1. `docker exec -it nifi bash`
2. `bin/nifi.sh set-single-user-credentials <username> <password>`
3. `exit`

## Usage
1. In the NiFi Operate Palette click *Upload Template*
2. Click to search, the xml file can be found in the repository by navigatating to the *etl-tracking-project/template/* directory. Choose the **tracking-pipeline.xml** file
3. In the Components Toolbar, drag the *Template* icon onto the grid
4. The uploaded template will display in an *Add Template* pop-up, click *ADD*
5. Right-click or in the Operate Palette, select the *Configuration* gear
6. Navigate to the *Controller Services* tab and add the *DistributedMapCacheServer* controller service and enable it with the default configuration
7. Enable the remaining 5 controller services by clicking on the enable icon (lightning bolt)
8. Navigate into the **Ingest Data** process group and configure the following PutS3Object processor properties: *Access Key ID* and *Secret Access Key* with the respective *minio* credentials created in the installation steps
9. Right-click or in the Operate Palette, select the *Start* option, this starts all the processors
10. Navigate into the **Generate Data** process group and further into the **ONLY RUN ONCE** process group
11. Right-click on the *GenerateFlowfile* processor, called **KafkaDruidSupervisor**, enable and start it
   > The processor creates the Druid Kafka ingestion spec body and posts it to Druid.
   This lets druid run real-time jobs that will pull data from Kafka topic into druid nodes for segmenting.
   Wait-Notify logic was added to ensure that the Druid-Kafka supervisor schema is first posted on Druid so that it can start monitoring the Kafka topic as soon as data is added to the topic

* Druid database can be accessed at [Druid](http://localhost:8888/) 
* Minio is the chosen object store where the raw and transformed data is saved and can be accessed at [Minio](http://localhost:9002/), with the configured *minio* credentials
* Kafdrop was added as a nice to have, to visualize the topics and messages that are sent to Kafka. It can be accessed at [Kafdrop](http://localhost:9001/)
