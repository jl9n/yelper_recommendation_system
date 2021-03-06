# Yelper: A Collaborative Filtering Based Recommendation System

Chuan Sun 

[chuansun76 at gmail dot com]

[twitter.com/sundeepblue]


This README file describes several major component of the "Yelper", a business recommendation system built mainly in Python using Spark framework.

Below are some features of the "Yelper":

- Divide original business data by cities allows fine tuned and customized recommendation
- Matrix Factorization based recommendation using Spark MLlib
- User-business graph analysis using Spark GraphX in Scala
- Real-time user request handling using Spark Streaming and Apache Kafka
- User-business graph visualization using D3 and graph-tool library
- Functional webserver to recommend high rated stuff for users

Now let me introduce in detail how to reproduce everything!

# 1. Preprocessing

(1) Convert all user ids and business ids to integers. This made subsequent graph building a lot easier.

(2) Split the entire business data into smaller subsets by city. Obtained 9 major cities:

- us_charlotte 
- us_lasvegas
- us_madison
- us_phoenix
- us_pittsburgh
- us_urbana_champaign
- canada_montreal
- germany_karlsruhe
- uk_edinburgh

All necessary util functions can be found here: ./rating_data_utils.py

Run this command:

> $ spark-submit ./parse_ratingdata_for_major_cities.py





# 2. Network analysis for user-business graph

## Extract connected components using Spark GraphX

Since there is no Python support for GraphX, I wrote the code in Scala. Note, the scala code has to be built using "sbt". Make sure the GraphX library is properly configured in file "./spark_graphx_analysis/config.sbt".

Source code: "./spark_graphx_analysis/src/main/scala/YelpUserBusinessGraphAnalysis.scala"

Below are the commands to run the graph analysis:

- $ cd /Users/sundeepblue/Bootcamp/allweek/week9/capstone/spark_graphx_analysis
- $ sbt package
- $ spark-submit --master local --class "YelpUserBusinessGraphAnalysis" target/scala-2.11/simple-project_2.11-1.0.jar

The file is saved to "businessid_to_indegree.csv"






# 3. Build MF-based recommendation models for 9 major cities

Run this command to prepare mf based model for each major city:

> $ python mf_based_recommendation_trainer.py






# 4. Build real-time user request handler using Spark Streaming and Apache Kafka

The purpose here is to simulate continuous user request handling.

### STEP 0: Start Zookeeper and Kafka server
Note that kafka zookeeper default port is 2181 not 9092! And, the zookeeper server and kafka server should be started in two separate terminals.

- $ cd /Users/sundeepblue/Bootcamp/allweek/week9/capstone/kafka/kafka_2.11-0.10.0.1
- $ bin/zookeeper-server-start.sh config/zookeeper.properties
- $ bin/kafka-server-start.sh config/server.properties

### STEP 1: Create Kafka topic
- $ bin/kafka-topics.sh --create --zookeeper localhost:2181 --topic user-request-topic --partitions 1 --replication-factor 1

### STEP 2: Launch Spark Streaming

Note that this command should also be run in a new terminal. Use port 2181 and use this topic: "user-request-topic"

- $ cd /Users/sundeepblue/Bootcamp/allweek/week9/capstone
- $ spark-submit --packages org.apache.spark:spark-streaming-kafka-0-8_2.11:2.0.0 ./handle_user_requests_streaming.py

### STEP 3: Produce user requests (TBD)
Note, do not specify port in KafkaProducer()!

- $ cd /Users/sundeepblue/Bootcamp/allweek/week9/capstone
- $ python ./user_requests_producer.py





# 5. Build user-business graph for D3 visualization

The purpose here is to generate .js file using all the nodes and edges in the user-business graph, such that I can load it and visualize graphs in web server.

See file: 

> ./build_nodes_and_edges_js_for_d3_visualization.py





# 6. Finally, how to run local web server to recommend something?

This web server was built using:

* Spark
* Flask
* cherrypy
* Python paste

### How to launch the web server?

- $ cd /Users/sundeepblue/Bootcamp/allweek/week9/capstone/webserver
- $ unset PYSPARK_DRIVER_PYTHON
- $ spark-submit server.py

### Sample web server URLs

recommendation for user '10081786' in city 'us_charlotte':

- http://0.0.0.0:5000/user_id/10081786/city/us_charlotte/top/30/keywords/restaurants

recommendation for user '10033545' in city 'us_madison':

- http://0.0.0.0:5000/user_id/10033545/city/us_madison/top/300/keywords/food

users-businesses social network in Madison, USA:

- http://0.0.0.0:5000/network

The javascript file "./webserver/static/data/generated_nodes_and_edges_from_json_us_madison.js" was programmatically generated by python code "./build_nodes_and_edges_js_for_d3_visualization.py"


### The most important files for the server

- ./webserver/app.py (contains code to interact with Spark, recommend businesses, etc)
- ./webserver/server.py (how to make spark work with flask)
- ./templates/map.html (contain Google Map API calling)




# 7. Future works

- More graph analysis
    - Graph pagerank analysis using GraphX
    - Community discovery (similar to Facebook social network)
- Improve recommendation
    - Content-based recommendation
    - Clustering all businesses
    - Extract object from business photos using Convolutional Neural Network
- Code
    - Redirect spark execution log to txt file
    - Add try/except to handle potential exceptions in codes
    - Add more comments
    - Add test cases for critical logics
- Google Map based web page
    - Fine tune the webpage to support more features


