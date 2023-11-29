---
layout: post
title: Kafka with docker-compose
featured-img: shane-rounce-205187
category: [Data Engineering]
summary: How to construct and execute a zookeeper and kafka cluster with docker-compose.
---

# Description
- Construct Zookeeper ensemble and Kafka cluster composed of n brokers, and jupyter-lab using docker-compose.
- Get experience of kafka producer & consumer service.

# Introduction
(1) You're recommended to use machine with 16GB memory or above. Also, you need linux shell environment with docker and docker-compose V2 installed. (I worked with MacOS system)

(2) Docker images are based on RedHat linux (CentOS 8) with bash shell, which is similar to real-world servers. Open-source versions are like below.
- jdk 11 (java 11)
- zookeeper 3.6.2
- zk-web latest
- kafka 2.7.0
- scala 2.13.0
- cmak(Cluster Manager for Apache Kafka) 3.0.0.5

(3) All files related to this practice are in my github. If you're used to Dockerfile, you can revise the image with Dockerfile in github.
                                     
[https://github.com/hjben/docker](https://github.com/hjben/docker)   
Sub-folders related on: zookeeper, kafka, jupyter-lab

(4) This practice uses shell script files in folder named by _docker-script_, where are sub-folder of each folder in github. This shell files get some parameter from user, and then constructs cluster and executes some files needed. docker-compose.yml file also be generated automatically in same path, and deleted when the docker-compose is down.

(5) The docker images are in my docker hub. You can download them with _docker pull_ command.

[https://hub.docker.com/search?q=hjben&type=image](https://hub.docker.com/search?q=hjben&type=image)

(6) If you have any problems, questions or bugs when using codes, just contact me.

# Docker container construction
### 1. Download docker image
The version of image maybe changed up, with update of the open-source version.
```
docker pull hjben/jupyter-lab:latest
docker pull hjben/cmak:3.0.0.5
docker pull hjben/kafka:2.7.0
docker pull hjben/zk-web:latest
docker pull hjben/zookeeper:3.6.2
```

### 2. Generate docker container.
(1) Zookeeper cluster must be created before kafka, because kafka cluster uses zookeeper cluster when they run.

(2) Download the shell script in _zookeeper/docker-script_ folder at the github and move them to the path where docker commands are available.

(3) With _./compose-up.sh_ command, docker network and containers are generated. parameters must be entered behind the command with one blank (space-bar) and arranged by the order below. The command will not be executed when lack of number of parameters or wrong input type detected.
- zookeeper_version: Version of zookeeper (3.8.1 and 3.6.2 is available now)
- (The # of ensemble) servers: The number of ensemble servers (odd integer between 1 and 5)
- web_user: User of zk-web (admin is recommended)
- web_password: Password of zk-web (admin is recommended)
- data_path: Host path for saving zookeeper znode data
- log_path: Host path for saving zookeeper log

e.g.
```
./compose-up.sh 3.6.2 3 admin admin /tmp/zookeeper /tmp/zookeeper_logs
```

(4) If created container, zookeeper initialization and execution of zookeeper is done. zk-web server starts after zookeeper run  as background process. Zookeeper ensemble between the containers also be set with automatically sensing another zookeeper containers.

(5) CLI(Command Line Interface) used above is occupied by foreground process of zk-web server, Open a new CLI(=shell) at host. TDownload the shell script in _kafka/docker-script_ folder at the github and move them to the path where docker commands are available. The two docker-script should be placed in different path to prevent confusion.

(6) With _./compose-up.sh_ command, docker network and containers are generated. parameters must be entered behind the command with one blank (space-bar) and arranged by the order below. The command will not be executed when lack of number of parameters or wrong input type detected.
- kafka_version: Version of kafka (2.7.0 is available now)
- cmak_version: Version of cmak (3.0.0.5 is available now)
- (The # of) servers: The number of broker servers (integer between 1 and 5)
- zookeeper_connect: Address of zookeeper ensemble (if composed of 3 servers, address is 10.0.3.1:2181,10.0.3.1:2182,10.0.3.1:2183)
- external_ip: external host ip to access kafka (if accessing in host local, external ip is 127.0.0.1)
- jupyter_workspace_path: Host path for saving jupyter lab workspace data
- data_path: Host path for saving kafka metadata
- log_path: Host path for saving kafka log

e.g.
```
./compose-up.sh 2.7.0 3.0.0.5 3 10.0.3.1:2181,10.0.3.1:2182,10.0.3.1:2183 127.0.0.1 /Users/Shared/workspace/docker-ws/kafka-notebook /tmp/kafka /tmp/kafka_logs﻿
```

(7) When created kafka container, cmak web server also be run automatically.

(8) There are volumes mounted on host path in each container for logs or data backup. This volumes are used for keep the data when docker containers are broken. Host path must be set with compatibility of your test environment, through some parameters of _./compose-up.sh_ command. The host paths you'll use are be made by yourself, preparing when they may not be automatically generated on the host.

(9) Execute _./compose-down.sh_ command if you want to destroy all containers.

(10) _./compose-up.sh_ command may mis-correct the generated docker-compose.yml file constructing another containers without compose-down command. So if you want to change the number of hadoop slaves, you're recommended to run _./compose-down.sh_ first.

(11) Zookeeper and kafka are composed of separate network, referencing different docker-compose.yml content.

# Zookeeper
### 1. Execute zookeeper
(1) zookeeper ensemble server and zk-web server are run when the container created, you could save the time to enter zookeeper execution command.

(2) The address of web ui is _localhost:18080_ and ID/PW are known as you entered (maybe admin/admin) when making the zookeeper container generation. But there're no problems using guest, without login.

### 2. Search zk-web page
<img src ="https://raw.githubusercontent.com/hjben/hjben.github.io/master/_img/kafka-cluster/zk-web.png" alt="zk-web">

(1) There're some lists of zookeeper cluster in the page, but with the first time of accessing ui, no list would be found.

(2) Enter the zookeeper url you want and press Go button, you could see cluster details page as below. 

(3) If you want to see zookeeper cluster made just a moment ago, the input url is _zoo1:2181,zoo2:2181,zoo3:2181_ (if the number of ensemble server is 3).

<img src ="https://raw.githubusercontent.com/hjben/hjben.github.io/master/_img/kafka-cluster/zk-web-detail.png" alt="zk-web-detail">

# Kafka
Kafka needs zookeeper, they must be executed with the well-running zookeeper cluster. At this time, we'll use the zookeeper cluster made above.

### 1. Execute kafka
(1) Open a new CLI(=shell) as much as the # of brokers (e.g. 3 CLIs). The reason of multi-opening is kafka broker service will be started as foreground process. Then, move into the path docker scripts are in.

(2) You can start kafka service at the broker server by entering each command below at the new CLI separately. The parameter of kafka-start.sh file is number of broker server. There're 1 to the number of broker servers you entered when creating kafka server.
```
./kafka-start.sh 1
./kafka-start.sh 2
./kafka-start.sh 3
...
```

(3) The port number of broker is 9092, but it's just used for internal connection. External port of broker is 9092 + (# of broker server).

(4) When started it, broker information is registered on zookeeper znode. The znode path is /kafka. They could be found in zk-web like below.

<img src ="https://raw.githubusercontent.com/hjben/hjben.github.io/master/_img/kafka-cluster/zk-web-kafka.png" alt="zk-web-kafka">

### 2. Search CMAK
CMAK (Cluster Manager for Apache Kafka) is a open-source kafka web ui made by Yahoo.

(1) CMAK server is also executed when the container created, no execution command is needed to run.

(2) The address of CMAK web ui is _localhost:9000_. You can see page like below if access CMAK.

<img src ="https://raw.githubusercontent.com/hjben/hjben.github.io/master/_img/kafka-cluster/kafka-cmak.png" alt="kafka-cmak">

(3) Register a kafka cluster by pressing Cluster -> Add Cluster button.
- Cluster Name has no naming rule, so it could be any words.
- Cluster Zookeeper Hosts is {zookeeper ensemble address}/kafka 입니다.   
(10.0.3.1:2181,10.0.3.1:2182,10.0.3.1:2183/kafka if you composed ensemble with 3 server)

- Accessment of CMAK to zookeeper is external access at the position of zookeeper so external zookeeper address is necessary.   
_10.0.3.1_ is network gateway address of zookeeper cluster, and 2181~218x is external port of each zookeeper container.

- Set Kafka Version as 2.4.0. It is earlier kafka version than you use, but no problems raised when using them.

- Other check-box below would be set like below. And then, press the Save button at the bottom.

﻿<img src ="https://raw.githubusercontent.com/hjben/hjben.github.io/master/_img/kafka-cluster/kafka-cmak-add.png" alt="kafka-cmak-add">

﻿(6) After registering cluster, you can manage the kafka cluster in CMAK web ui.(Checking message status or deletion of topic, etc.).   

(7) There're some brokers and topics in the cluster if the cluster is correctly registered.

﻿<img src ="https://raw.githubusercontent.com/hjben/hjben.github.io/master/_img/kafka-cluster/kafka-cmak-list.png" alt="kafka-cmak-list">

### 3. Test kafka
Test would be done with some shell files in each broker. They're in $KAFKA_HOME/bin folder.

(1) Open two new CLIs at host. Each CLI will be used as producer and consumer.

(2) In each CLI, access the kafka broker shell with _docker exec -it kafka{kafka_broker_number} bash_ command. Any broker to access will be fine. From now on, all test codes will be run in the broker shell.

e.g.
```
docker exec -it kafka1 bash﻿
```

(3) Default JMX_PORT is 9999, but it's occupied by broker. Thus you should select another port. And producer and consumer are run in foreground, different JMX_PORT would be necessary if they're running in same broker server.

(4) Create a topic named by test-topic, with replication 1 and partition 1. You must push the zookeeper address with _--zookeeper_ option.
```
JMX_PORT=9998 kafka-topics.sh --zookeeper 10.0.3.1:2181,10.0.3.1:2182,10.0.3.1:2183/kafka --replication-factor 1 --partitions 1 --topic test-topic --create
```

(5) Execute kafka producer connected to test-topic. A message would be sent by command below. Enter the internal addresses of all brokers in --broker-list option.
```
JMX_PORT=9998 kafka-console-producer.sh --broker-list kafka1:9092,kafka2:9092,kafka3:9092 --topic test-topic
```

(6) Execute kafka consumer connected to test-topic in another CLI. It receive messages producer sent. All broker's internal address is needed in --bootstrap-server option.
```
JMX_PORT=9998 kafka-console-consumer.sh --bootstrap-server kafka1:9092,kafka2:9092,kafka3:9092 --topic test-topic --from-beginning
```
﻿
### 4. Execute jupyter lab
(1) Open a new CLI(=shell) at host to start jupyter lab. Then, move into the path docker scripts are in.

(2) Enter _/jupyter-start.sh_ command to run jupyter lab.

(3) Copy the URL in the console and access the jupyter lab service. Copy and Paste the address started with 127.0.0.1 to your web browser. Token is changing value every time you run.

e.g.
```
http://127.0.0.1:8888/lab?token=e76b79faeabb826699a80166e39c5627390923dc24e053aa
```

### 5. Test kafka with python
(1) Create a new notebook in the jupyter lab.

(2) Install kafka-python package.
```
!pip install kafka-python (in notebook) or
docker exec -it jupyter-lab bash -c "pip install kafka-python" (in host CLI)
```

<a href="https://raw.githubusercontent.com/hjben/hjben.github.io/master/_file/kafka-cluster/kafka-sample.zip">kafka-sample.zip</a>

(3) Using the sample code in this zip file, you can test kafka producer and consumer with python.

(4) The sample data and code is able to use after unzipping the attached file and uploading jupyter lab.

(5) Kafka producer and consumer will be started by running each sample files.

﻿<img src ="https://raw.githubusercontent.com/hjben/hjben.github.io/master/_img/kafka-cluster/kafka-producer.png" alt="kafka-producer">
﻿<img src ="https://raw.githubusercontent.com/hjben/hjben.github.io/master/_img/kafka-cluster/kafka-consumer.png" alt="kafka-consumer">

(6) If producer sends a message, the producer code will exit. Consumer, on the other hand, searches kafka queue permanently.

(7) You could find if producer sends a message, consumer catches and print it.
