# Spark Streaming Graceful Shutdown

![Graceful Shutdown Logo](graceful_shutdown_logo.png)

## Table of Contents

<details open>
<summary><b>(click to expand or hide)</b></summary>
<!-- MarkdownTOC -->

1. [Introduction](#1-introduction)  
   1.1. [Advantages of graceful shutdown](#11-advantages-of-graceful-shutdown)    
   1.2. [Different ways to implement graceful shutdown](#12-different-ways-to-implement-graceful-shutdown)  
2. [Prerequisites](#2-prerequisites)  
3. [Build, Deploy and Run the Spark Streaming Application](#3-build-deploy-and-run-the-spark-streaming-application)  
    3.1. [Build and Deploy the Application](#31-build-and-deploy-the-application)  
    3.2. [Run the Spark Streaming application using the Socket source](#32-run-the-spark-streaming-application-using-the-socket-source)  
    3.3. [Run Spark Streaming application using the Kafka source](#33-run-spark-streaming-application-using-the-kafka-source)  
    3.4. [Stop the Spark Streaming Job gracefully](#34-stop-the-spark-streaming-job-gracefully)  
4. [References](#4-references)  
5. [Authors](#5-authors)  

<!-- /MarkdownTOC -->
</details>

## 1) Introduction

Spark Streaming jobs are **long-running jobs** and their tasks need to be executed **7*24** hours, but there are cases 
like **upgrading the code** or **updating the Spark configuration**, where the program needs to be actively stopped. 

Due to a non-graceful shutdown, the streaming application may stop abruptly in the middle of processing a batch, 
leading to potential data loss or data duplication. To avoid such issues, it is crucial to ensure that the streaming 
application is gracefully shutdown. 

### 1.1) Advantages of graceful shutdown

There are several advantages of using graceful shutdown in Spark Streaming:

1) Prevent data loss
2) Reduce errors

### 1.2) Different ways to implement graceful shutdown

There are several ways to gracefully shutdown a Spark Streaming application:

<span style="color:red">
1) Explicitly calling the JVM Shutdown Hook in the driver program <br/>
2) Using spark.streaming.stopGracefullyOnShutdown = true <br/>
</span>

<span style="color:green">
3) Use an external approach to control internal program shutdown <br/>
4) Expose the interface to the outside world and provide the shutdown function
</span>

## 2) Prerequisites 

| Component | Version |
|-----------|---------|
| Java      | 1.8     |
| Scala     | 2.11.12 |
| Spark     | 2.4.7   |
| Kafka     | 2.5.0   |

## 3) Build, Deploy and Run the Spark Streaming Application

### 3.1) Build and Deploy the Application

#### 3.1.1) Download the "spark_streaming_graceful_shutdown" application

```sh
$ git clone https://github.com/rangareddy/spark_streaming_graceful_shutdown.git
$ cd spark_streaming_graceful_shutdown
```

#### 3.1.2) Build the `spark_streaming_graceful_shutdown` application

> Before building the application, ensure that you update the Spark and other component's library versions according to your cluster version.

```sh
$ mvn clean package -DskipTests
```

#### 3.1.3) Log in to the Spark gateway node, such as "mynode.host.com", and create the application deployment directory.

```sh
$ ssh username@mynode.host.com
$ mkdir -p /apps/spark/spark_streaming_graceful_shutdown
$ chmod 755 /apps/spark/spark_streaming_graceful_shutdown
```

#### 3.1.4) Copy the JAR file to the Spark gateway node

```sh
$ scp target/spark-streaming-graceful-shutdown-1.0.0-SNAPSHOT.jar username@mynode.host.com:/apps/spark/spark_streaming_graceful_shutdown
```

### 3.2) Run the Spark Streaming application using the Socket source

#### 3.2.1) Launch the SocketStream by running the following command in the shell:

```sh
$ nc -lk 9999
```

#### 3.2.2) Copy the run script file to the Spark gateway node such as "mynode.host.com"

```sh
$ scp run_spark_streaming_socket_graceful_shutdown_app.sh username@mynode.host.com:/apps/spark/spark_streaming_graceful_shutdown
```

#### 3.2.3) Log in to the Spark gateway node, such as "mynode.host.com", and execute the application.

Before running the application, please ensure the following:

1. Verify that you have the necessary permissions to execute the script.
2. Update the values of `HOST_NAME` and `PORT` in the script to match your specific configuration.

Once you have confirmed these details, you can proceed to run the application.

```sh
sh /apps/spark/spark_streaming_graceful_shutdown/run_spark_streaming_socket_graceful_shutdown_app.sh
```

The script will prompt you to enter an option based on the desired approach:

> If you want to use the marker filesystem approach, you should provide the HDFS file system path as the input. 
For example, you can enter "/apps/spark/streaming/shutdown/spark_streaming_kafka_marker_file". It is important to 
use a unique path for each application.

> If you want to use the HTTP service approach, you should provide the Jetty service port as the input. For example, 
you can enter "3443". Ensure that you use a unique port number for each application.

After entering the required information, the script will proceed accordingly.

### 3.3) Run Spark Streaming application using the Kafka source

#### 3.3.1) Create the Kafka topic and verify the kafka topic is created or not.

```sh
kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 3 --topic test-topic
kafka-topics --list --bootstrap-server localhost:9092
```

#### 3.3.2) Produce messages to the Kafka topic.

```sh
kafka-console-producer --bootstrap-server localhost:9092 --topic test-topic
```

(or)

```sh
kafka-console-producer --bootstrap-server localhost:9092 --topic test-topic < my_file.txt
```

#### 3.3.3) Consume messages from the Kafka topic.

```sh
kafka-console-consumer --bootstrap-server localhost:9092 --topic test-topic --from-beginning
```

#### 3.3.4) Copy the run script file to the Spark gateway node such as "mynode.host.com"

```shell
$ scp run_spark_streaming_socket_graceful_shutdown_app.sh username@mynode.host.com:/apps/spark/spark_streaming_graceful_shutdown
```

#### 3.3.5) Login to spark gateway node (for example mynode.host.com) and run the application.

Before running the application, please verify the following:

a) Ensure that you have the necessary permissions to run the script. If you encounter any permission issues, contact 
your system administrator or check the file permissions to ensure that you have execute permissions.

b) Update the `BOOTSTRAP_SERVERS` value in the script. The `BOOTSTRAP_SERVERS` variable should be set to the list of 
Kafka broker endpoints in the format <host1:port1>,<host2:port2>,.... 
Replace <host1:port1>,<host2:port2>,... with the actual Kafka broker endpoints for your cluster. 

For example:

```sh
BOOTSTRAP_SERVERS="kafka1:9092,kafka2:9092,kafka3:9092"
```

c) Ensure that the Kafka broker endpoints are correct and accessible from the machine where you are running the script.

```sh
sh /apps/spark/spark_streaming_graceful_shutdown/run_spark_streaming_kafka_graceful_shutdown_app.sh
```

The script will prompt you to enter an option based on the desired approach:

> If you want to use the marker filesystem approach, you should provide the HDFS file system path as the input. 
For example, you can enter "/apps/spark/streaming/shutdown/spark_streaming_kafka_marker_file". It is important to use a 
unique path for each application.

> If you want to use the HTTP service approach, you should provide the Jetty service port as the input. For example, 
you can enter "3443". Ensure that you use a unique port number for each application.

After entering the required information, the script will proceed accordingly.

### 3.4) Stop the Spark Streaming Job gracefully

#### 3.4.1) Stop the Spark Streaming Job using Shutdown Hook Approach

**Client Mode:**

a) There are multiple ways to send the shutdown hook signal. One of the mostly used approach is by pressing the Ctrl+C.

**Cluster Mode:**

a) Go to the Spark UI and find out the driver hostname from the Executors tab.

    -   Access the Spark UI by navigating to the URL of your Spark cluster (e.g., http://spark-master:4040).
    
    -   Click on the Executors tab in the Spark UI.

    -   Look for the row that represents the driver, and note down the hostname of the driver.

b) Login to the Driver host and find out the Application Master process id.

    -   Open a terminal or SSH into the host machine where the Spark driver is running.

    -   Run the following command to find the process id (PID) of the Application Master:

        ```shell
        jps -m | grep "spark.deploy.master.Master"
        ```

    -   The above command will display the process id and other information of the Spark Application Master. Note down the process id.

c) Kill the process id.

    -   Run the following command to kill the Application Master process:

        ```sh
        kill -9 <process_id>`
        ```

    -   Replace <process_id> with the actual process id you obtained in the previous step.

#### 3.4.2) Stop the Spark Streaming Job using Shutdown Signal Approach

**Client Mode:**

a) There are multiple ways to send the shutdown hook signal. One of the mostly used approach is by pressing the Ctrl+C.

**Cluster Mode:**

a) Go to the Spark UI and find out the driver hostname from the Executors tab.

    -   Access the Spark UI by navigating to the URL of your Spark cluster (e.g., http://spark-master:4040).
    
    -   Click on the Executors tab in the Spark UI.

    -   Look for the row that represents the driver, and note down the hostname of the driver.

b) Login to the Driver host and find out the Application Master process id.

    -   Open a terminal or SSH into the host machine where the Spark driver is running.

    -   Run the following command to find the process id (PID) of the Application Master:

        ```shell
        jps -m | grep "spark.deploy.master.Master"
        ```

    -   The above command will display the process id and other information of the Spark Application Master. Note down the process id.

c) Kill the process id.

    -   Run the following command to kill the Application Master process:

        ```sh
        kill -9 <process_id>`
        ```

    -   Replace <process_id> with the actual process id you obtained in the previous step.

#### 3.4.3) Stop the Spark Streaming Job using Shutdown Marker Approach

In this approach, stopping the Spark Streaming job is same for both client and cluster mode.

a) Collect the marker file path specified while running the Spark application. For example, marker file path is 

`/apps/spark/streaming/shutdown/spark_streaming_kafka_marker_file`.

b) Create an hdfs directory

```sh
# hdfs dfs -mkdir -p /apps/spark/streaming/shutdown
```

c) Create a marker file under an HDFS directory

```sh
# hdfs dfs -touch /apps/spark/streaming/shutdown/spark_streaming_kafka_marker_file
```

> Make sure submitted user has access to the above path.

d) Once the application is successfully stopped then marker file will be deleted automatically.

#### 3.4.4) Stop the Spark Streaming Job using Shutdown HTTP Approach

In this approach, stopping the Spark Streaming job is same for both client and cluster mode.

a) Collect the HTTP port specified while running the Spark application. For example the http port is `3443`.

b) To collect the HTTP service path for your Spark application, you can follow these steps:

    -   Open the Spark UI by accessing the URL provided by your Spark cluster (e.g., http://<spark_master_host>:4040).
    -   Go to the "Executors" tab in the Spark UI.
    -   Find the driver executor and note the value under the "Host" column. This is the driver host.
    -   Once you have the driver host, you can construct the HTTP service path using the format http://<driver_host>:<port>/shutdown/<app_name>.

c) Run the following url in the browser with the updated driver_host. For example.

`http://localhost:3443/shutdown/SparkStreamingKafkaGracefulShutdownHttpApp`

> You need to replace localhost with driver_host and correct application name.
> Ensure that the Spark application is running and the HTTP service is set up correctly for the shutdown to be executed successfully.

You will see the following output in the browser after graceful shutdown.

```sh
The Spark Streaming application has been successfully stopped.
```

## 4) References

- [Spark Streaming Graceful Shutdown - Part1](https://community.cloudera.com/t5/Community-Articles/Spark-Streaming-Graceful-Shutdown-Part1/ta-p/366958)
- [Spark Streaming Graceful Shutdown - Part2](https://community.cloudera.com/t5/tkb/workflowpage/tkb-id/CommunityArticles/article-id/6748)

## 5) Authors

- [@rangareddy](https://github.com/rangareddy)