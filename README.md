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
7. <b>config-server</b>: microservice to load all config-server-repository configuration <br>
8. <b>common-config</b>: common config across services
9. <b>common-util</b>: common utility across services
10. <b>common-util</b>: common utility across services
11. <b>elastic</b>: contains elastic sub module<br>
	- <b>elastic-config</b>: holds elastic config<br>
	- <b>elastic-index-client</b>: create and manages elastic index<br>
	- <b>elastic-model</b>: holds elastic models<br>
	- <b>elastic-query-client</b>: holds client config to perform operation on elastic(two implementation check @Primary)<br>

12. <b>elastic-query-service</b>: elastic search query implementation
13. <b>kafka-to-elastic-service</b>: send elastic data to kafka service
14. <b>elastic-query-web-client</b>: UI(thymleaf) to search test `http://localhost:8184/elastic-query-web-client`
15. <b>elastic-query-service-common</b>: Hold reactive common 
16. <b>elastic-query-web-client-common</b>:  hold elastic client
17. <b>reactive-elastic-query-service</b>: hold reactive service 
18. <b>reactive-elastic-query-web-client</b>: hold reactive query controller <b> to run:</b> Run Config server, then `docker-compose -f common.yml -f elastic_cluster.yml up` then reactive-elastic-query-service then finally reactive-elastic-query-web-client and search text like "java" on  `http://localhost:8184/reactive-elastic-query-web-client/query-by-text` 

# keycloak setup:
- add keycloak_authorization_server.yml with admin/admin user and password
- create postgress user: <b>keycloak</b> and password: <b>keycloak</b> with admin privilege and create database <b>keycloak</b>(see config in keycloak_authorization_server.yml file) 
- run `docker-compose keycloak_authorization_server.yml up`
- `http://localhost:8081/auth/` then click administration console -> put admin/admin user/pass
- `http://localhost:8081/auth/admin/master/console/#/create/realm` create realm name: <b>microservices-realm</b> (click master left top corner to add realm)
- <b>microservices-realm</b>: <br>
	 - Roles -> add Role -> add 3 roles app_admin_role, app_user_role and app_super_user_role <br>
	 - Groups -> new -> add 3 group app_admin_group, app_user_group, app_super_user_group corresponding to Role<br>
	 - Now click on each group created above and assign role to corresponding group -> Role Mapping: app_super_user_group - app_super_user_role, app_admin_group - app_admin_role and app_user_group - app_user_role <br>
	 - Users -> add user -> add 3 user app-user, admin-user and app-super-user with password(Credentials tab): <b>perm</b> <br>
	 - Now click each user to map(join) corresponding group, Users -> Groups - join corresponding group, app-user- app_user_group, app_admin_group and app-super-user - group-app_super_user_role <br>
	 - Clients -> add Client -> Client ID=elastic-query-web-client and Client Protocol=openid-connect -> create -> setting -> Access Type=creadentials ,Enabled=ON, Standard Flow Enabled=ON, Direct Access Grants Enabled=ON,Service Accounts Enabled=ON Valid Redirect URIs: http://localhost:8184/elastic-query-web-client/login/oauth2/code/keycloak and http://localhost:8184/elastic-query-web-client, Base URL= http://localhost:8184/elastic-query-web-client, Web Origins = http://localhost:8184, we will use 74b63390-7a43-4c82-a400-6eb48490ab5b (credential tab) for configuration in code then Mappers tab -> create -> Name =microservices-groups, Mapper Type= Group Membership,Token Claim Name= groups and Full group path = OFF -> save, then create another mapper -> Name= elastic-query-service, Mapper Type = Audience, Included Custom Audience=elastic-query-service -> save, then create another ->
	 Name = client-id, Mapper Type=User Session Note, User Session Note=clientID, Token Claim Name=clientID, Claim JSON TypeString ->save, then create another ->
	 Name = client-ip, Mapper Type=User Session Note, User Session Note=clientAddress, Token Claim Name=clientAddress, Claim JSON TypeString ->save <br>
	 - ClientScopes -> create -> Name = app-user-role, Protocol-openid-connect -> save then Scope tab on same page -> assign role app-user-role <br>
	  - ClientScopes -> create -> Name = app-admin-role, Protocol-openid-connect -> save then Scope tab on same page -> assign role app-admin-role <br>
	  - ClientScopes -> create -> Name = app_super_user_role, Protocol-openid-connect -> save then Scope tab on same page -> assign role app_super_user_role <br>
	  - Clients->elastic-query-web-client-> Client Scopes tab -> Default Client Scopes - select all scopes(app_super_user_role,app-admin-role and app-user-role) and assign <br> <br>
	  
- Clients -> add Client -> Client ID=elastic-query-web-client-2 (for signle sign up) and Client Protocol=openid-connect -> create -> setting -> Access Type=creadentials ,Enabled=ON, Standard Flow Enabled=ON, Direct Access Grants Enabled=ON,Service Accounts Enabled=ON Valid Redirect URIs: http://localhost:8185/elastic-query-web-client/login/oauth2/code/keycloak and http://localhost:8185/elastic-query-web-client, Base URL= http://localhost:8185/elastic-query-web-client, Web Origins = http://localhost:8185 (creadential 21bfb1af-36a0-42f9-a261-90cc19107f17) ,
 then Mappers tab -> create -> Name =microservices-groups, Mapper Type= Group Membership,Token Claim Name= groups and Full group path = OFF -> save, 
then create another mapper -> Name= elastic-query-service, Mapper Type = Audience, Included Custom Audience=elastic-query-service -> save, 
then create another ->
	 Name = client-id, Mapper Type=User Session Note, User Session Note=clientID, Token Claim Name=clientID, Claim JSON TypeString ->save, 
then create another ->
	 Name = client-host, Mapper Type=User Session Note, User Session Note=clientHost, Token Claim Name=clientHost, Claim JSON TypeString ->save <br>
then create another ->
	 Name = client-ip, Mapper Type=User Session Note, User Session Note=clientAddress, Token Claim Name=clientAddress, Claim JSON TypeString ->save <br>
	then Scope tab on same page -> assign role app_super_user_role <br>
	  - Clients->elastic-query-web-client-> Client Scopes tab -> Default Client Scopes - select all scopes(app_super_user_role,app-admin-role and app-user-role) and assign <br> <br>
	  
- Clients -> add Client -> Client ID=elastic-query-service and Client Protocol=openid-connect -> create -> setting -> Access Type=creadentials ,Enabled=ON, Standard Flow Enabled=ON, Direct Access Grants Enabled=ON,Service Accounts Enabled=ON Valid Redirect URIs: http://localhost:8183/elastic-query-service/login/oauth2/code/keycloak and http://localhost:8183/elastic-query-service, Base URL= http://localhost:8183/elastic-query-service, Web Origins = http://localhost:8183 (credential d25b101b-30fb-44b8-87a9-bff4a0bf46f7),
 then Mappers tab -> create -> Name =microservices-groups, Mapper Type= Group Membership,Token Claim Name= groups and Full group path = OFF -> save, 
then create another mapper -> Name= kafka-streams-service, Mapper Type = Audience, Included Custom Audience=kafka-streams-service -> save, 
hen create another mapper -> Name= analytics-service, Mapper Type = Audience, Included Custom Audience=analytics-service -> save, 
then create another ->
	 Name = client-id, Mapper Type=User Session Note, User Session Note=clientID, Token Claim Name=clientID, Claim JSON TypeString ->save, 
then create another ->
	 Name = client-host, Mapper Type=User Session Note, User Session Note=clientHost, Token Claim Name=clientHost, Claim JSON TypeString ->save <br>
then create another ->
	 Name = client-ip, Mapper Type=User Session Note, User Session Note=clientAddress, Token Claim Name=clientAddress, Claim JSON TypeString ->save <br>
	then Scope tab on same page -> assign role app_super_user_role <br>
	  - Clients->elastic-query-web-client-> Client Scopes tab -> Default Client Scopes - select all scopes(app_super_user_role,app-admin-role and app-user-role) and assign <br>
	  - Finally can see config `http://localhost:8081/auth/realms/microservices-realm/.well-known/openid-configuration`<br>
	  - http://localhost:8184/elastic-query-web-client <br> to search text


Note: when dockerize please replace all Clients urls with servicename instead on localhost as in container localhost will not work-<br>
http://elastic-query-web-client-1:8184/elastic-query-web-client/login/oauth2/code/keycloak<br>
http://elastic-query-web-client-2:8185/elastic-query-web-client/login/oauth2/code/keycloak<br>
http://elastic-query-service-1:8183/elastic-query-service/login/oauth2/code/keycloak<br>
also change keycloak port from 8081 to 9091 as 8081 is used by kafka steam service in docker<br>

and `mvn clean install -DskipTests`
then can `http://elastic-query-web-client-1:8184/elastic-query-web-client` provide login user name and pass eg. app-user/perm or admin-user/perm or app-super-user/perm then you can seach text based on authentication 
	  
	  
	  
	 
	 





## How to run
- clone the git repo <a href="https://github.com/ratneshvarma/event-driven-microservice-architecture.git">https://github.com/ratneshvarma/event-driven-microservice-architecture.git</a>
- run config-server service first (localhost:8888)
- `cd ./event-driven-microservice-architecture/docker-compose` 
- `docker-compose -f common.yml -f kafka_cluster.yml up` or (`docker-compose up` to run all yml inside folder) (to start kafka docker services otherwise application failed to run)
- run main application (TwitterToKafkaServiceApplication.run())

Note: If you can check each microservice logs using `docker logs <container-id>` (-f option show continues logs)
- `export ENCRYPT_KEY='Demo_Pwd!2020'`
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
- search index:- <br> curl --location --request GET 'http://localhost:9200/twitter-index/_search'<br>
- search by id:- <br> curl --location --request GET 'http://localhost:9200/twitter-index/_search?q=id:6546787733652269481'<br>
- query:-<br> curl --location --request POST 'http://localhost:9200/twitter-index/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
  "query": {
    "term": {
       "text": "test"
      }
   }
}'<br>
- curl --location --request POST 'http://localhost:9200/twitter-index/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
  "query": {
     "match": {
       "text": "test multi word"
      }
   }
}' <br>
- curl --location --request POST 'http://localhost:9200/twitter-index/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
  "query": {
     "term": {
       "text": "test multi word"                           
      }
   }
}' <br>
- curl --location --request POST 'http://localhost:9200/twitter-index/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
  "query": {
     "wildcard": {
       "text": "te*"                          
      }
   }
}' <br>
- curl --location --request POST 'http://localhost:9200/twitter-index/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
  "query": {
    "query_string": {
       "query": "text:te*"                          
      }
   }
}' <br>
- curl --location --request POST 'http://localhost:9200/twitter-index/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
  "from": 0,
  "size": 20,
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "text": "test"
          }
        },
        {
          "match": {
            "text": "word"
          }
        }
      ]
    }
  }
}' <br>
- <b>Elastic query service</b>:- `curl --location --request GET 'http://localhost:8183/elastic-query-service/documents'`<br> 
get byId  `curl --location --request GET 'http://localhost:8183/elastic-query-service/documents/6369722700539622078'` <br> version2- <br> curl --location --request GET 'http://localhost:8183/elastic-query-service/documents/6369722700539622078' \
--header 'Content-Type: application/vnd.api.v2+json'
- <b>swagger</b>: `http://localhost:8183/elastic-query-service/swagger-ui.html`



http://elastic-query-web-client-1:8184/elastic-query-web-client/login/oauth2/code/keycloak
http://elastic-query-web-client-2:8185/elastic-query-web-client/login/oauth2/code/keycloak
http://elastic-query-service-1:8183elastic-query-service/login/oauth2/code/keycloak

	 
    				 
   
    	
	 