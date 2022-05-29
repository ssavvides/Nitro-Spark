## Running Spark cluster inside Docker containers on multiple EC2 instances

First thing needed is to be able to run instances on the AWS management console. Once the access is establish simply launch three instances that will serve as the worker, master and driver nodes. Connect to each one and run the containers as before. This time we create a Docker daemon to be the swarm manager:
```
docker swarm init
```
This should give a command where we need to check via the AWS console that the private ip matches the one of the instance .
After running the ```docker swarm join``` we can now create a overlay-network that will have lined up the two containers of the two machines:
```
docker network create --driver overlay --attachable spark-overlay-net
```
Make sure the incoming and oucoming ports are open otherwise cannot bind the swarm deamon manager. Particularly creating a security group on AWS management console we can add inbounding rules and outbunding rules enabling the ports necessary for the connection.

Now we can conclude by running the containers over the network created, on the machines accordingly.
On the first master EC2 Instance run the master container connecting to the overlay network that we have just created:
```
docker run -it --name spark-master --network spark-overlay-net -p 8080:8080 davide0110/spark_master /bin/bash
```
On the second worker EC2 Instance erun the worker container as we have done for the master:
```
docker run -it --name spark-worker --network spark-overlay-net -p 8081:8081 davide0110/spark_worker /bin/bash
```
Multiple workers can be created with the same procedure, without the need to connect to a different port

On a third instance run the spark-submit container:
```
docker run -it --name spark-submit --network spark-overlay-net -p 4040:4040 davide0110/spark_submit /bin/bash
```
Now that we have the architecture setup and the terminal waiting for our actions we can launch a spark-shell or even use a spark-submit script to submit a spark application, on the third instance for example:
```
$SPARK_HOME/spark/bin/spark-shell --conf spark.executor.cores=3 --conf spark.executor.memory=6G --master spark://spark-master:7077
```
From there for example we can run an example program in the spark-shell, by replacing the paths, we will count how often a word occurs in a given file:
```
val file = sc.textFile("/tmp/data")
val counts = file.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_ + _)
counts.count()
```
A spark-submit command can also be used to submit a job to the spark cluster. For example:
```
./bin/spark-submit \
      --master spark://spark-master:7077 \
      examples/src/main/python/pi.py \
      1000
```
![Schermata 2022-05-18 alle 14 18 37_preview_rev_1](https://user-images.githubusercontent.com/43402963/169038099-ef157eff-54e0-42b8-9599-d67d2727c286.png)