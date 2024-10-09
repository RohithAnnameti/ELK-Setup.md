# ELK-Setup.md

# Update the packages
sudo apt update -y

# Install the java
sudo apt install openjdk-11-jdk -y

# Install Nginx
sudo apt install nginx -y

# Install Elastic search
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.15.2-amd64.deb
sudo dpkg -i elasticsearch-8.15.2-amd64.deb
sudo systemctl start elasticsearch
sudo systemctl status elastisearch

# Install Kibana
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.15.2-amd64.deb
sudo dpkg -i kibana-8.15.2-amd64.deb
sudo systemctl status kibana

# Install Logstash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install logstash
sudo systemctl enable logstash
sudo systemctl start logstash
sudo systemctl status logstash

# Install Filebeat
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.15.2-amd64.deb
sudo dpkg -i filebeat-8.15.2-amd64.deb

# Configure Elastic Search
sudo vi /etc/elasticsearch/elasticsearch.yml 
Uncomment Cluster.name, node.name and network.host and change the network.host to local host and uncomment http.port  
sudo systemctl start elasticsearch
sudo systemctl status elasticsearch

# Configure Kibana
sudo vi etc/kibana/kibana.yml -- uncomment server.host and server.port
sudo systemctl start kibana
sudo systemctl status kibana

# Adding htacess passwd
sudo apt-get install apache2-utils -y
sudo htpasswd -c /etc/nginx/htpasswd.users kibadmin --kibadmin is the username
sudo nano /etc/nginx/htpasswd.users - to check the uname and passwd.

# Configuring the nginx
sudo vim /etc/nginx/sites-available/default
