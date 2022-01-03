# event-driven-microservice-architecture
1. <b>twitter-to-kafka-service</b>: stream mock/actual twitter data (check config for mock  enable-mock-tweets: true) and comunicate with kafka admin(to create topic) and kafka procedure(to send messages)
2. <b>app-config-data</b>: common config module to read config values and provide across all modules using pom dependency
3. <b>docker-compose</b>: use to create kafka, services, config elastic docker(cluster/node(brocker)(.env hidden file) </br>
  	 - move into docker-compose directory `cd docker-compose` </br>
	 - run `docker-compose -f common.yml -f kafka_cluster.yml up` </br>
	 - you can use kafkacat tool to monitor kafka using network option to reach host machine(in our case its localhost) `docker run -it --network=host confluentinc/cp-kafkacat kafkacat -L -b localhost:19092` (-L = list, -b = broker) which return topics, brokers and partitions details</br>
	 - use `kafkacat -L -b localhost:19092` (if already activate network option, result same) e.g.- </br>
	     3 brokers:
  			 broker 2 at localhost:29092 (controller)
  			 broker 1 at localhost:19092
  			 broker 3 at localhost:39092
 			2 topics:
  				topic "__confluent.support.metrics" with 1 partitions:
   				 partition 0, leader 2, replicas: 2,3,1, isrs: 2,3,1
 				 topic "_schemas" with 1 partitions:
    				 partition 0, leader 2, replicas: 2,3,1, isrs: 2,3,1<br> and `docker run -it --network=host confluentinc/cp-kafkacat kafkacat -C -b localhost:19092 -t twitter-topic` to see the messages on twitter-topic e.g.- <br> % Reached end of topic twitter-topic [0] at offset 2<br>
̡?ߤ???????ģ????sit posuere sed ultricies congue Elasticsearch amet Lorem consectetuerએ??<br>
4. <b>kafka</b>: multi Module to hold model, admin and producer for kafa<br>
	 - <b>kafka-module</b>: holds model to communicate/exchange messages, we are using apache avro for this which read the file from resource/avro  and create java class in java main folder<br>
	 - <b>kafka-admin</b>: manages all activity related to kafka, example creating topics and retrying, addding checks etc<br>
	 - <b>kafka-producer</b>: Using apache kafka, communicate to kafka through our application, it provides template to send messages, and annotation to consume kafka topic and some listner<br>
5. <b>common-config</b>: common config to retry <br>
6. <b>config-server-repository</b>:Its git repo to have all config to make externalization (use `cd ./config-server-repository` then `git init` <br> or use (https://github.com/ratneshvarma/config-server-repository if using remote repo)
7. <b>config-server</b>:microservice to load all config-server-repository configuration <br>


## How to run
- clone the git repo <a href="https://github.com/ratneshvarma/event-driven-microservice-architecture.git">https://github.com/ratneshvarma/event-driven-microservice-architecture.git</a>
- run config-server service first (localhost:8888)
- `cd ./event-driven-microservice-architecture/docker-compose` 
- `docker-compose -f common.yml -f kafka_cluster.yml up` or (`docker-compose up` to run all yml inside folder) (to start kafka docker services otherwise application failed to run)
- run main application (TwitterToKafkaServiceApplication.run())

Note: If you can check each microservice logs using `docker logs <container-id>` (-f option show continues logs)
- export ENCRYPT_KEY='Demo_Pwd!2020'
- mvn clean install -DskipTest( or run config-server first then mvn clean install as context load test will fail)
- see config data http://localhost:8888/config-client/twitter_to_kafka
- `cd ./docker-compose` then `chmod +x check-config-server-started.sh` (its entrypoint of twitter to kafka service)
- `docker exec -it <container-id> /bin/bash` to go inside docker container
- for dockerise issue, first run `mvn clean install -DskipTests` then `docker-compose up` as docker will only create once mvn command used
- `docker-compose -f common.yml -f elastic_cluster.yml up` to run elasticsearch docker
- elasticsearch `docker-compose -f common.yml -f elastic_cluster.yml up` to check elasticsearch `curl --location --request GET 'http://localhost:9200'` (increase docker memory up to 4 gb) 
- create elaticsearch index:- <br>curl --location --request PUT 'http://localhost:9200/twitter-index' \
--header 'Content-Type: application/json' \
--data-raw '{
    "mappings": {
        "properties": {
            "userId": {
                "type": "long"
            },
            "id": {
                "type": "text",
                "fields": {
                    "keyword": {
                        "type": "keyword",
                        "ignore_above": 256
                    }
                }
            },
            "createdAt": {
                "type": "date",
		"format": "yyyy-MM-dd'\''T'\''HH:mm:ssZZ"
            },
            "text": {
                "type": "text",
                "fields": {
                    "keyword": {
                        "type": "keyword",
                        "ignore_above": 256
                    }
                }
            }
        }
    }
}'<br>
- create document in index:- <br> curl --location --request POST 'http://localhost:9200/twitter-index/_doc/1' \
--header 'Content-Type: application/json' \
--data-raw '{
"userId": "1",
"id": "1",
"createdAt": "2020-01-01T23:00:50+0000",
"text": "test multi word"
}'<br>

	 
    				 
   
    	
	 