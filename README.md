## Local Up & Running

We have used a computer with Ubuntu 22.04 installed.

### Downloading data

```bash
resources/download_data.sh
```

### Installing

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

  mongod
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

  We start it going to the main directory (`cd /opt/kafka`) and running `bin/kafka-server-start.sh config/server.properties`.

  * __Zookeeper:__

  ```bash
  wget https://dlcdn.apache.org/zookeeper/zookeeper-3.5.10/apache-zookeeper-3.5.10-bin.tar.gz
  tar xfvz apache-zookeeper-3.5.10-bin.tar.gz
  sudo mv apache-zookeeper-3.5.10-bin.tar.gz /opt/zookeeper
  ```

  We start it going to the Kafka directory (`cd /opt/kafka`) and running `bin/zookeeper-server-start.sh config/zookeeper.properties`.

### Creating a topic in Kafka

```bash
 bin/kafka-topics.sh \
  --create \
  --bootstrap-server localhost:9092 \
  --replication-factor 1 \
  --partitions 1 \
  --topic flight_delay_classification_request
```

### Importing the distance records to MongoDB

```bash
./resources/import_distances.sh
```

### Training and saving the model

```bash
 python3 resources/train_spark_mllib_model.py .
 ```

### Running Flight Predictor

We open IntelliJ with `intellij-idea-ultimate`. We open our project in the `flight_prediction` directory, install the necessary plugins and build the program. When that is finished, we can run the program.

### Starting the prediction request web app

```bash
python3 resources/web/predict_flask.py
```

Go to `http://localhost:5000/flights/delays/predict_kafka` to try the working application.

## Docker&Docker-Compose Up & Running

`docker-compose up -d`

### Kafka

Ìnside container (`docker-compose exec kafka bash`):

 * Create the topic
 
```bash
kafka-topics.sh --create \
  --bootstrap-server kafka:9092 \
  --replication-factor 1 \
  --partitions 1 \
  --topic flight_delay_classification_request
```

### MongoDB

Inside container (`docker-compose exec mongodb bash`)

 * Import the distance records

 ```bash
 cd flight_prediction
 ./resources/import_distances.sh

 ```

