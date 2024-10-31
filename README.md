# Stock Market Data Pipeline

## Overview
This project demonstrates a real-time data streaming pipeline using **Apache Kafka**, **Python**, and **Amazon S3**. The pipeline involves setting up Kafka and Zookeeper on an **Amazon EC2** instance, using a Kafka **Producer** to send data from a CSV file to a Kafka topic, using a Kafka **Consumer** to read data from the Kafka topic and write it to an **Amazon S3** bucket in JSON format.

## Features
- **Kafka Setup Scripts**: Step-by-step instructions to install and configure Kafka and Zookeeper on an EC2 instance.
- **Kafka Producer Notebook**: A Jupyter Notebook (`Kafka_Producer.ipynb`) that reads data from a CSV file and sends it to a Kafka topic at regular intervals.
- **Kafka Consumer Notebook**: A Jupyter Notebook (`Kafka_Consumer.ipynb`) that consumes messages from a Kafka topic and writes them to Amazon S3.
- **Data Streaming Simulation**: Simulates real-time data streaming using sample data.
- **AWS Integration**: Demonstrates integration with AWS services like EC2 and S3.

## Usage
### A. Setting Up the Environment
1. **Clone the Repository**
    ```
    git clone https://github.com/yourusername/your-repo-name.git
    cd your-repo-name
    ```

2. **Launch an EC2 Instance**
- Use an Amazon Linux 2 AMI.
- Configure the security group to allow inbound traffic on ports `22` (SSH), `9092` (Kafka), and `2181` (Zookeeper).

3. **Install Java on EC2**
- SSH into your EC2 instance and run:
    ```
    sudo yum update -y
    sudo yum install -y java-1.8.0-openjdk
    ```

### B. Installing and Configuring Kafka
1. **Download and Extract Kafka**
- Navigate to the `scripts` directory on your EC2 instance and execute:
    ```
    wget https://downloads.apache.org/kafka/3.3.1/kafka_2.12-3.3.1.tgz
    tar -xvf kafka_2.12-3.3.1.tgz
    ```

2. **Configure Kafka Server Properties**
- Edit the `config/server.properties` file in the Kafka directory.
- Set `advertised.listeners` to your EC2 instance's public IP address:
    ```properties
    advertised.listeners=PLAINTEXT://<your_public_ip>:9092
    ```
- Replace `<your_public_ip>` with the actual public IP address of your EC2 instance.

3. **Start Zookeeper**
    ```
    cd kafka_2.12-3.3.1
    bin/zookeeper-server-start.sh config/zookeeper.properties
    ```

4. **Start Kafka Broker**
- Open a new terminal session to your EC2 instance and run:
    ```
    export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"
    cd kafka_2.12-3.3.1
    bin/kafka-server-start.sh config/server.properties
    ```

5. **Create a Kafka Topic**
- In another terminal session:
    ```
    cd kafka_2.12-3.3.1
    bin/kafka-topics.sh --create --topic demo_test --bootstrap-server <your_public_ip>:9092 --replication-factor 1 --partitions 1
    ```

### C. Running the Kafka Producer
1. **Install Dependencies**
- On your local machine or a separate environment where you'll run the producer:
    ```
    pip install kafka-python pandas
    ```

2. **Prepare the Data**
- Ensure the CSV file `indexProcessed.csv` is located in the `data` directory.
- This file should contain the data you want to stream.

3. **Configure the Producer**
- Open `Kafka_Producer.ipynb`.
- Update the `bootstrap_servers` parameter with your EC2 instance's public IP.

4. **Run the Producer Notebook**
- Execute all cells in the notebook.
- The producer will send a random row from the CSV file to the Kafka topic every second.

### D. Running the Kafka Consumer
1. **Install Dependencies**
    ```
    pip install kafka-python s3fs
    ```

2. **Configure AWS Credentials**
- Ensure AWS credentials with appropriate permissions are configured:
    - Use `aws configure` to set up your credentials.
    - Alternatively, set environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.

3. **Configure the Consumer**
- Open `Kafka_Consumer.ipynb`.
- Update the `bootstrap_servers` parameter.
- Update the S3 bucket name in the notebook.

4. **Run the Consumer Notebook**
- Execute all cells in the notebook.
- The consumer will write each message to a JSON file in the specified S3 bucket.

## Prerequisites
- **Amazon EC2 Instance**: For hosting Kafka and Zookeeper.
- **Java JDK 1.8**: Required on the EC2 instance to run Kafka.
- **Python 3.x**: For running the producer and consumer scripts.
- **Jupyter Notebook**: To open and run `.ipynb` files.
- **AWS CLI**: For configuring AWS credentials on the consumer machine.
- **AWS S3 Bucket**: An existing S3 bucket to store consumed messages.
- **Kafka-Python Library**: Install using `pip install kafka-python`.
- **s3fs Library**: Install using `pip install s3fs`.
- **Pandas Library**: Install using `pip install pandas`.

## Input
- **Kafka Topic** of your choice.
- **CSV Data File** containing sample data to be streamed.

## Output
- **JSON Files in S3**: Each consumed message is saved as a JSON file in your specified S3 bucket.
    - Files are named `stock_market_<count>.json`

## Notes
- **Security Groups**: Ensure the EC2 instance's security group allows inbound traffic on the necessary ports.
- **Public vs. Private IP**: Use the **public IP address** of your EC2 instance when configuring clients outside the EC2 instance.
- **AWS Permissions**: The AWS credentials used by the consumer must have `PutObject` permissions for the S3 bucket.
- **Data Schema**: Adjust the producer and consumer code if your data schema differs from the sample.
- **Resource Cleanup**: Remember to terminate the EC2 instance and delete S3 objects to avoid unnecessary charges.
- **Error Handling**: For production use, implement robust error handling and logging.
- **Kafka Topic Configuration**: You can adjust replication factor and partitions based on your needs.