# Step 1: Deploy Elasticsearch Instance
Launch an EC2 instance with Ubuntu, choosing t2.medium as the instance type
# Update the package list: 
sudo apt-get update  
# Install the default JDK and JRE:
sudo apt install default-jdk default-jre -y
# Add Elasticsearch’s GPG key and install necessary packages:
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add 
sudo apt-get install apt-transport-https
# Add Elasticsearch’s APT repository:
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee –a /etc/apt/sources.list.d/elastic-7.x.list
# Update the package list again:
sudo apt-get update -y
# Install Elasticsearch:
sudo apt-get install elasticsearch
# Configure Elasticsearch by editing the elasticsearch.yml file:
sudo vi /etc/elasticsearch/elasticsearch.yml
Set the network.host to the private IPv4 address of your instance.
Save and exit the file.
![image](https://github.com/user-attachments/assets/c4771e0b-9f75-416b-9bba-6dc3b9e3428d)
# Start Elasticsearch and check its status:
sudo systemctl start elasticsearch 
sudo systemctl status elasticsearch
# Verify the cluster is running:
curl http://<Private_IP>:9200
# ![image](https://github.com/user-attachments/assets/421342f2-ff88-4a7e-9a7a-2ae44406d0a3)

# Step 2: Deploy Logstash and Kibana Instance
Launch another EC2 instance with Ubuntu, naming it logstashkibana, and choose t2.medium.
SSH into the instance and update the package list
sudo su
sudo apt-get update
# Install the default JDK and JRE:
sudo apt install default-jdk default-jre -y
# Add Elasticsearch’s GPG key and install necessary packages:
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add 
sudo apt-get install apt-transport-https
# Add Elasticsearch’s APT repository:
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee –a /etc/apt/sources.list.d/elastic-7.x.list
# Update the package list again:
sudo apt-get update -y
# Install Kibana:
sudo apt-get install kibana
# Configure Kibana by editing the kibana.yml file:
sudo vi /etc/kibana/kibana.yml
Set server.host to the private IP of the logstashkibana instance.
Set elasticsearch.hosts to the private IP of the Elasticsearch instance.
Save and exit the file.
![image](https://github.com/user-attachments/assets/bb04058c-e89e-4388-b0f4-86d7e302d6d9)
# Start Kibana and check its status:
sudo systemctl start kibana 
sudo systemctl status kibana
# Verify Kibana is running by checking the logs:
sudo tail -f /var/log/kibana/kibana.log
# Access Kibana in a web browser using <PUBLIC_IP>:5601
# Step 3: Install Logstash
sudo apt-get install logstash
# Create a configuration file for Apache logs:
cd /etc/logstash/conf.d/ 
sudo vi apache.conf
# Add the appropriate configuration, setting the hosts field to the private IP of the Elasticsearch instance.
# Save and exit the file.
input {
  beats {
    port => "5044"
  }
}

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
}

output {
  elasticsearch {
    hosts => ["http://172.31.88.220:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
  stdout {
    codec => rubydebug
  }
}

![image](https://github.com/user-attachments/assets/5bbaf72b-fe72-4f89-9c89-107af238e126)
# Start Logstash and check its status:
sudo systemctl start logstash 
sudo systemctl status logstash
# Verify Logstash is running by checking the logs:
sudo tail -f /var/log/logstash/logstash-plain.log

# Step 4: Deploy the Client Instance
Launch a new EC2 instance (this will be the client instance) and SSH into it.
# Update the package list:
sudo apt-get update
# Install Apache:
sudo apt-get install apache2 -y
# Install Filebeat:
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.17.6-amd64.deb
sudo dpkg -i filebeat-7.17.6-amd64.deb
# Configure Filebeat by editing the filebeat.yml file:
sudo vi /etc/filebeat/filebeat.yml
Set the hosts field to the private IP of the logstashkibana instance.
Save and exit the file.
![image](https://github.com/user-attachments/assets/a92029d3-830b-4b7d-85ff-0284aea470c1)
![image](https://github.com/user-attachments/assets/0e953201-afb6-4c15-ba86-c01a0e58f3f1)
# Set up Filebeat and enable necessary modules:
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["<elasticsearch_IP>:9200"]'
sudo filebeat modules enable system
sudo filebeat modules enable apache
sudo systemctl restart filebeat.service
sudo filebeat test output
# Step 5: Visualize Apache Logs in Kibana
Access Kibana in a web browser using <PUBLIC_IP>:5601.
Navigate to the “Discover” tab and create a new index pattern for the Apache logs.
Once the index pattern is created, return to the “Discover” tab to visualize the data.
