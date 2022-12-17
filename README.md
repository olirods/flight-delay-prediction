## Up & Running (local)

These steps have been successfully reproduced in a system with Ubuntu 22.04 installed.

### 1. Downloading data

```bash
resources/download_data.sh
```

### 2. Installing services

First, we need to set an initial ENV variable indicating the path of this cloned repository, with `export PROJECT_HOME=/path/path`

 * __JDK 1.8:__ `sudo apt install openjdk-8-jdk`

Then, we set the ENV variable to indicate the path to our Java installation directory with `export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64`

 * __Intellij__: `sudo snap install intellij-idea-ultimate --classic`
 * __Python__: I had it already installed, but to assure the 3.7 version, I made use of `asdf`:

 ```bash
  asdf plugin add python
  asdf install python 3.7.9
  asdf local python 3.7.9
 ```

 Requirements also installed with `pip install -r requirements.txt`

 * __SBT:__ following official installation instructions:

 ```bash
  sudo apt-get update
  sudo apt-get install apt-transport-https curl gnupg -yqq
  echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | sudo tee /etc/apt/sources.list.d/sbt.list
  echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt_old.list
  curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | sudo -H gpg --no-default-keyring --keyring gnupg-ring:/etc/apt/trusted.gpg.d/scalasbt-release.gpg --import
  sudo chmod 644 /etc/apt/trusted.gpg.d/scalasbt-release.gpg
  sudo apt-get update
  sudo apt-get install sbt
```

 * __Mongo:__ following official installation instructions:

 ```bash
  wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
  echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
  wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2_amd64.deb
  dpkg -i libssl1.1_1.1.1f-1ubuntu2_amd64.deb
  sudo apt-get update

  sudo mkdir -p /data/db
  sudo chown -R mongodb:mongodb /data/db
 ```

  * __Spark:__ we had to find the 3.1.2 version throughout the Internet. Following this steps:

  ```bash
  wget https://archive.apache.org/dist/spark/spark-3.1.2/spark-3.1.2-bin-hadoop3.2.tgz
  tar xfvz spark-3.1.2-bin-hadoop3.2.tgz
  sudo mv spark-3.1.2-bin-hadoop3.2 /opt/spark
  ```

  Then, we set the ENV variable to indicate our Spark installation folder with `export SPARK_HOME=/opt/spark`

  * __Scala:__ I had it already installed, but to assure the 2.12 version, I made use of `asdf`:

 ```bash
  asdf plugin add scala
  asdf install python 2.12.3
  asdf local python 2.12.3
 ```

  * __Kafka:__ 

  ```bash
  wget https://archive.apache.org/dist/kafka/3.0.0/kafka_2.12-3.0.0.tgz
  tar xfvz kafka_2.12-3.0.0.tgz 
  sudo mv kafka_2.12-3.0.0 /opt/kafka
  ```

  * __Zookeeper:__

  ```bash
  wget https://dlcdn.apache.org/zookeeper/zookeeper-3.5.10/apache-zookeeper-3.5.10-bin.tar.gz
  tar xfvz apache-zookeeper-3.5.10-bin.tar.gz
  sudo mv apache-zookeeper-3.5.10-bin.tar.gz /opt/zookeeper
  ```
  
### 3. Starting the database

We can do it with just:

```
mongod
```

We can check it's up with `service mongod status`.

### 4. Starting Kafka and Zookeeper

We need first to change the working directory to the Kafka installation one (`cd /opt/kafka`).

We begin running Zookeeper:

```
bin/zookeeper-server-start.sh config/zookeeper.properties
```

And then Kafka:

```
bin/kafka-server-start.sh config/server.properties
```

### 5. Creating a topic in Kafka

```bash
 cd /opt/kafka

 bin/kafka-topics.sh \
  --create \
  --bootstrap-server localhost:9092 \
  --replication-factor 1 \
  --partitions 1 \
  --topic flight_delay_classification_request
```

![image](https://user-images.githubusercontent.com/49884623/207953370-cbaba93b-2324-40d1-bcfc-f5741b8fdc6f.png)


### 6. Importing the distance records to MongoDB

```bash
./resources/import_distances.sh
```

![image](https://user-images.githubusercontent.com/49884623/207953466-b9fb74ae-96cf-4b35-a8aa-634fc9199103.png)


### 7. Training and saving the model

```bash
 python3 resources/train_spark_mllib_model.py .
 ```
 
![image](https://user-images.githubusercontent.com/49884623/207961819-77df56f4-7f7a-470f-b406-93201ad28019.png)

### 8. Running the prediction job

#### Option 1: IntelliJ
We open IntelliJ with `intellij-idea-ultimate`. We open our project in the `flight_prediction` directory, install the necessary plugins and build the program. When that is finished, we can run the program.

![Screenshot from 2022-12-14 00-09-23](https://user-images.githubusercontent.com/49884623/207467679-eda2c8a6-04c7-4561-8aaf-a8810b418400.png)

#### Option 2: Spark Submit

We'll first compile the code and build a JAR file using sbt:

```
cd flight_prediction
sbt package
```

Then a new .jar package will have been created at `flight_prediction/target/scala-2.12/flight_prediction_2.12-0.1.jar`.

Now, it's necessary to create a Spark master and a Spark worker to run the job. We go to the Spark installation directory (`cd /opt/spark`). We create the master:

```
./sbin/start-master.sh
```

If you go now to `http://localhost:8080` you can see info about this master node and you will see its URL at the top of the page. That is the one we need to create a worker for this master:

```
./sbin/start-worker.sh <master-spark-URL>
```

If you reload the Spark master page, you will see the worker associated.

![image](https://user-images.githubusercontent.com/49884623/208167788-77af5910-a3ba-4cb5-bcd7-cc6b99eb76d2.png)

Finally, we are aready to launch the prediction job with spark-submit. Just use this command:

```
./bin/spark-submit \
  --class es.upm.dit.ging.predictor.MakePrediction \
  --master <master-spark-URL> \
  --deploy-mode cluster \
  --packages org.mongodb.spark:mongo-spark-connector_2.12:3.0.1,org.apache.spark:spark-sql-kafka-0-10_2.12:3.1.2 \
  flight_prediction/target/flight_prediction_2.12-0.1.jar
```

If successfully, you will see then in the Spark master webpage that there is a new running driver for our job:

![image](https://user-images.githubusercontent.com/49884623/208169174-3d4c7ef0-7b4b-4d7c-b661-0cd5f57ff867.png)

### 9. Starting the prediction request web app

```bash
python3 resources/web/predict_flask.py
```

Go to `http://localhost:5000/flights/delays/predict_kafka` to try the working application.

![Screenshot from 2022-12-14 00-11-58](https://user-images.githubusercontent.com/49884623/207467632-c3097051-4864-4274-8df6-6c4adeb64637.png)

### 10. Checking the predictions records in the db

You can enter to the MongoDB shell with `mongosh` and then follow:

```
> use agile_data_science;
> db.flight_delay_classification_response.find()
```

![Screenshot from 2022-12-14 00-22-32](https://user-images.githubusercontent.com/49884623/207467761-65938dfb-c3a3-4cae-aacf-584aa395c5de.png)

## Docker&Docker-Compose Up & Running

To run all the system you just need to do `docker-compose up -d`. Before that, make sure you have all the necessary generated files, the ones you got from following steps 1 (downloaded data), 7 (trained models) and 8-2 (.jar package file). 

### Creating the topic in Kafka

Ìnside container (`docker-compose exec kafka bash`):
 
```bash
kafka-topics.sh --create \
  --bootstrap-server kafka:9092 \
  --replication-factor 1 \
  --partitions 1 \
  --topic flight_delay_classification_request
```

### Importing the distance records to MongoDB

Inside container (`docker-compose exec mongodb bash`)

 ```bash
 cd flight_prediction
 ./resources/import_distances.sh

 ```

### Running the prediction job with Spark-Submit

Inside Spark worker container (`docker-compose exec spark-worker bash`)

```bash
./bin/spark-submit \
  --class es.upm.dit.ging.predictor.MakePrediction \
  --master spark://spark-master:7077 \
  --deploy-mode cluster \
  --packages org.mongodb.spark:mongo-spark-connector_2.12:3.0.1,org.apache.spark:spark-sql-kafka-0-10_2.12:3.1.2 \
  data/flight_prediction_2.12-0.1.jar
```

