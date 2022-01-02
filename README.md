# event-driven-microservice-architecture
1. <b>twitter-to-kafka-service</b>: stream mock/actual twitter data (check config for mock  enable-mock-tweets: true)
2. <b>app-config-data</b>: common config module to read config values and provide across all modules using pom dependency
3. <b>docker-compose</b>: use to create kafka doker(cluster/node(brocker) </br>
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
    				 partition 0, leader 2, replicas: 2,3,1, isrs: 2,3,1
    	
	 