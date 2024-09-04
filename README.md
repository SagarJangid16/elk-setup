
# ELK Stack Setup Guide

This guide provides step-by-step instructions for provisioning a virtual machine, installing, and configuring the ELK Stack (Elasticsearch, Logstash, Kibana), setting up a sample application, creating dashboards in Kibana, and securing and optimizing the stack. Follow these steps to successfully deploy the ELK Stack on a cloud-based virtual machine.

## 1. Provisioning the Virtual Machine

### Step 1: Choose a Free Cloud Service
Select a cloud provider that offers a free tier, such as AWS. The steps provided here use AWS Free Tier.

### Step 2: Create and Launch a Virtual Machine

1. **Create an Account**: Sign up for AWS if you don’t have an account. Log in to the AWS Management Console.
2. **Launch an EC2 Instance**:
   - Go to the EC2 Dashboard and click on "Launch Instance."
   - Choose **Ubuntu Server 22.04 LTS** (free tier eligible).
   - Select the `t2.micro` instance type (free tier eligible).
   - Configure instance details (default settings are fine).
   - Add 8 GB of storage.
   - Configure the security group:
     - Allow SSH (port 22), HTTP (port 80), Kibana (port 5601), Elasticsearch (port 9200), and Logstash (port 5044).
   - Review and launch the instance.
   - Create or select an existing key pair for SSH access.

### Step 3: Connect to the Instance

Use SSH to connect to the instance:

```bash
ssh -i /path/to/your-key.pem ubuntu@<your-ec2-public-ip>
```

## 2. ELK Stack Installation and Configuration

### Step 4: Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 5: Install Java (Prerequisite for Elasticsearch and Logstash)

```bash
sudo apt install openjdk-11-jdk -y
```

### Step 6: Install and Configure Elasticsearch

1. **Install Elasticsearch**:
   - Add the Elasticsearch GPG key and repository:

     ```bash
     wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
     sudo apt install apt-transport-https
     echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list
     ```

   - Install Elasticsearch:

     ```bash
     sudo apt update && sudo apt install elasticsearch -y
     ```

   - Start and enable Elasticsearch:

     ```bash
     sudo systemctl start elasticsearch
     sudo systemctl enable elasticsearch
     ```

2. **Configure Elasticsearch**:
   - Open the configuration file:

     ```bash
     sudo nano /etc/elasticsearch/elasticsearch.yml
     ```

   - Modify the following settings for optimization:

     ```yaml
     network.host: 0.0.0.0
     http.port: 9200
     ```


   - Restart Elasticsearch:

     ```bash
     sudo systemctl restart elasticsearch
     ```

### Step 7: Install and Configure Logstash

1. **Install Logstash**:

   ```bash
   sudo apt install logstash -y
   ```

2. **Configure Logstash**:
   - Create a configuration file:

     ```bash
     sudo nano /etc/logstash/conf.d/logstash.conf
     ```

   - Example configuration:

     ```bash
     input {
       file {
         path => "/var/log/syslog"
         start_position => "beginning"
       }
     }

     filter {
       grok {
         match => { "message" => "%{SYSLOGLINE}" }
       }
     }

     output {
       elasticsearch {
         hosts => ["http://localhost:9200"]
         index => "system-logs-%{+YYYY.MM.dd}"
       }
     }
     ```

   - Start and enable Logstash:

     ```bash
     sudo systemctl start logstash
     sudo systemctl enable logstash
     ```

### Step 8: Install and Configure Kibana

1. **Install Kibana**:

   ```bash
   sudo apt install kibana -y
   ```

2. **Configure Kibana**:
   - Open the Kibana configuration file:

     ```bash
     sudo nano /etc/kibana/kibana.yml
     ```

   - Set:

     ```yaml
     server.host: "0.0.0.0"
     ```

   - Start and enable Kibana:

     ```bash
     sudo systemctl start kibana
     sudo systemctl enable kibana
     ```

## 3. Set Up a Sample Application to Generate Logs

### Step 9: Create a Sample Application (Optional)

1. **Create a Simple Web Server**:

   You can use Python’s built-in HTTP server as an example:

   ```bash
   mkdir ~/sample-app
   cd ~/sample-app
   echo "<h1>Welcome to ELK Stack</h1>" > index.html
   python3 -m http.server 8080
   ```

   This will generate access logs that can be monitored.

2. **Generate Logs**:

   Generate logs by accessing the application in your browser:

   ```bash
   curl http://localhost:8080
   ```

## 4. Create Dashboards in Kibana

### Step 10: Access Kibana and Create Dashboards

1. **Access Kibana**:

   In your browser, go to `http://<your-ec2-public-ip>:5601`.

2. **Set up an index pattern for logs**:
   - Go to `Management > Stack Management > Index Patterns`.
   - Create an index pattern for `system-logs-*`.

3. **Create Dashboards**:
   - Go to the Dashboard section.
   - Create visualizations:
     - **Error Rate Over Time**: Use a line graph to show the number of error logs over time.
     - **Request Logs by Status Code**: Use a pie chart to show the distribution of HTTP status codes.

## 5. Secure and Optimize the ELK Stack

### Step 11: Secure the ELK Stack

1. **Set Up Firewall Rules**:

   Use UFW to allow necessary traffic:

   ```bash
   sudo ufw allow OpenSSH
   sudo ufw allow 5601/tcp
   sudo ufw allow 9200/tcp
   sudo ufw allow 5044/tcp
   sudo ufw enable
   ```

2. **Basic Authentication**:

   Implement basic authentication for Kibana and Elasticsearch if necessary, or restrict access via security groups in AWS.

### Step 12: Monitor and Optimize

1. **Monitor Resource Usage**:

   Use tools like `htop`, `top`, to monitor resource usage.

2. **Optimize Elasticsearch**:

   Adjust the JVM heap size and disable unnecessary plugins to reduce memory usage.

## 6. Cleanup (Optional)

### Step 13: Terminate the EC2 Instance

If the setup is no longer needed, terminate the EC2 instance to avoid any potential charges.

---

