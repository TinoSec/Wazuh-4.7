# Wazuh-4.7
Instalación Wazuh 4.7

# Installing prerequisites

Some extra packages are needed for the installation

```bash
sudo apt-get install apt-transport-https zip unzip lsb-release curl gnupg
```

# Installing GPG KEY
Install the GPG key

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
```
Add the repository

```bash
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
```
Update the package information

```bash
sudo apt update
```
Install Wazuh manager

```bash
apt-get -y install wazuh-manager=4.7.5-1
```
Enable and start the Wazuh manager service.

```bash
systemctl daemon-reload
systemctl enable wazuh-manager
systemctl start wazuh-manager
```
Run the following command to verify the Wazuh manager status.

```bash
systemctl status wazuh-manager
```
# Install Filebeat
Install filebeat package
```bash
apt-get -y install filebeat
```
Download the preconfigured Filebeat configuration file.

```bash
curl -so /etc/filebeat/filebeat.yml https://packages.wazuh.com/4.7/tpl/wazuh/filebeat/filebeat.yml
```
Edit the /etc/filebeat/filebeat.yml configuration file and replace the following value "hosts".
Replace it with your Wazuh indexer address accordingly.

###### ejemplo de filebeat.yml



Create a Filebeat keystore to securely store authentication credentials.

```bash
filebeat keystore create
```

Add the default username and password admin:admin to the secrets keystore.

```bash
echo admin | filebeat keystore add username --stdin --force
echo admin | filebeat keystore add password --stdin --force
```

Download the alerts template for the Wazuh indexer.

```bash
curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/v4.7.5/extensions/elasticsearch/7.x/wazuh-template.json
chmod go+r /etc/filebeat/wazuh-template.json
```
Install the Wazuh module for Filebeat.

```bash
curl -s https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.3.tar.gz | tar -xvz -C /usr/share/filebeat/module
```
# Deploying certificates
Note Make sure that a copy of the wazuh-certificates.tar file, created during the initial configuration step, is placed in your working directory.Note Make sure that a copy of the wazuh-certificates.tar file, created during the initial configuration step, is placed in your working directory.

```bash
NODE_NAME=node-1
```
Create folder for certificate, and uncompress certificates to created folder. 
```bash
mkdir /etc/filebeat/certs
tar -xf ./wazuh-certificates.tar -C /etc/filebeat/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./root-ca.pem
mv -n /etc/filebeat/certs/$NODE_NAME.pem /etc/filebeat/certs/filebeat.pem
mv -n /etc/filebeat/certs/$NODE_NAME-key.pem /etc/filebeat/certs/filebeat-key.pem
chmod 500 /etc/filebeat/certs
chmod 400 /etc/filebeat/certs/*
chown -R root:root /etc/filebeat/certs
```
# Starting the Filebeat service
Enable and start the Filebeat service.

```bash
systemctl daemon-reload
systemctl enable filebeat
systemctl start filebeat
```
Run the following command to verify that Filebeat is successfully installed.

```bash
filebeat test output
```
Expand the output to see an example response.

##############################################################################






```bash
mkdir /etc/elasticsearch/certs/ca -p
cp -R ~/certs/ca/ ~/certs/elasticsearch/* /etc/elasticsearch/certs/
chown -R elasticsearch: /etc/elasticsearch/certs
chmod -R 500 /etc/elasticsearch/certs
chmod 400 /etc/elasticsearch/certs/ca/ca.* /etc/elasticsearch/certs/elasticsearch.*
rm -rf ~/certs/ ~/certs.zip
```

Enable and start the Elasticsearch service

```bash
systemctl daemon-reload
systemctl enable elasticsearch
systemctl start elasticsearch
```

Generate credentials for all the Elastic Stack pre-built roles and users

```bash
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
```
To check that the installation was made successfully, run the following command replacing *localhost* with your server ip and *<elastic_password>* with the password generated in the previous step for elastic user

```bash
curl -XGET https://localhost:9200 -u elastic:<elastic_password> -k
```

Example:

*curl -XGET https://192.168.1.1:9200 -u elastic:GeneratedPassword -k*

Correct output

<details>
  <summary>Output Elasticsearch Curl</summary>
  
  ```json
  {
    "name" : "elasticsearch",
    "cluster_name" : "elasticsearch",
    "cluster_uuid" : "44FdoqYaQxaYMtN_Ge7_dQ",
    "version" : {
      "number" : "7.17.12",
      "build_flavor" : "default",
      "build_type" : "deb",
      "build_hash" : "e3b0c3d3c5c130e1dc6d567d6baef1c73eeb2059",
      "build_date" : "2023-07-20T05:33:33.690180787Z",
      "build_snapshot" : false,
      "lucene_version" : "8.11.1",
      "minimum_wire_compatibility_version" : "6.8.0",
      "minimum_index_compatibility_version" : "6.0.0-beta1"
    },
    "tagline" : "You Know, for Search"
  }
```
</details>

Your configuration file should look like this:
[elasticsearch.yml](https://github.com/TinoSec/Wazuh-4.5-Install/blob/main/elasticsearch.yml)

# Install Wazuh-Server
Install GPG key
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
```
Add repository

```bash
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
```
Update packages

```bash
sudo apt update
```
Install Wazuh Package

```bash
apt-get install wazuh-manager
```
Enable and start the Wazuh manager service

```bash
systemctl daemon-reload
systemctl enable wazuh-manager
systemctl start wazuh-manager
```
Run the following command to check if the Wazuh manager is active

```bash
systemctl status wazuh-manager
```

# Install Filebeat
Install Filebeat package

```bash
apt-get install filebeat=7.17.12
```
Download the pre-configured Filebeat config file used to forward Wazuh alerts to Elasticsearch

```bash
curl -so /etc/filebeat/filebeat.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/filebeat_all_in_one.yml
```
Download the alerts template for Elasticsearch

```bash
curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/4.5/extensions/elasticsearch/7.x/wazuh-template.json
chmod go+r /etc/filebeat/wazuh-template.json
```
Download the Wazuh module for Filebeat

```bash
curl -s https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.2.tar.gz | tar -xvz -C /usr/share/filebeat/module
```
*Edit the file /etc/filebeat/filebeat.yml and add the following line:*

*output.elasticsearch.password: <elasticsearch_password>*

*Also edit output.elasticsearch.hosts value with your server ip  *

Replace elasticsearch_password with the previously generated password for elastic user.

Copy the certificates into /etc/filebeat/certs/

```bash
cp -r /etc/elasticsearch/certs/ca/ /etc/filebeat/certs/
cp /etc/elasticsearch/certs/elasticsearch.crt /etc/filebeat/certs/filebeat.crt
cp /etc/elasticsearch/certs/elasticsearch.key /etc/filebeat/certs/filebeat.key
```
Enable and start the Filebeat service

```bash
systemctl daemon-reload
systemctl enable filebeat
systemctl start filebeat
```

Test filebeat configuration

```bash
filebeat test output
```
Correct configuration output

<details>
  <summary>Output Filebeat Test</summary>
  
  ```
  {
elasticsearch: https://192.168.1.1:9200...
  parse url... OK
  connection...
    parse host... OK
    dns lookup... OK
    addresses: 192.168.1.1
    dial up... OK
  TLS...
    security: server's certificate chain verification is enabled
    handshake... OK
    TLS version: TLSv1.3
    dial up... OK
  talk to server... OK
  version: 7.17.12
  }
```
</details>

Your configuration file should look like this:
[filebeat.yml](https://github.com/TinoSec/Wazuh-4.5-Install/blob/main/filebeat.yml)

# Install Kibana

```bash
apt-get install kibana=7.17.12
```
Copy the Elasticsearch certificates into the Kibana configuration folder

```bash
mkdir /etc/kibana/certs/ca -p
cp -R /etc/elasticsearch/certs/ca/ /etc/kibana/certs/
cp /etc/elasticsearch/certs/elasticsearch.key /etc/kibana/certs/kibana.key
cp /etc/elasticsearch/certs/elasticsearch.crt /etc/kibana/certs/kibana.crt
chown -R kibana:kibana /etc/kibana/
chmod -R 500 /etc/kibana/certs
chmod 440 /etc/kibana/certs/ca/ca.* /etc/kibana/certs/kibana.*
```
Download the Kibana configuration file

```bash
curl -so /etc/kibana/kibana.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/kibana_all_in_one.yml
```
*Edit the /etc/kibana/kibana.yml file*
*elasticsearch.password: <elasticsearch_password>*

Create the /usr/share/kibana/data directory

```bash
mkdir /usr/share/kibana/data
chown -R kibana:kibana /usr/share/kibana
```
Install the Wazuh Kibana plugin. The installation of the plugin must be done from the Kibana home directory as follows

```bash
cd /usr/share/kibana
sudo -u kibana /usr/share/kibana/bin/kibana-plugin install https://packages.wazuh.com/4.x/ui/kibana/wazuh_kibana-4.5.2_7.17.12-1.zip
```
Link Kibana's socket to privileged port 443

```bash
setcap 'cap_net_bind_service=+ep' /usr/share/kibana/node/bin/node
```
Enable and start the Kibana service

```bash
systemctl daemon-reload
systemctl enable kibana
systemctl start kibana

```
Your configuration file should look like this:
[kibana.yml](https://github.com/TinoSec/Wazuh-4.5-Install/blob/main/kibana.yml)



Access the web interface using the password generated during the Elasticsearch installation process

URL: https://<wazuh_server_ip>

user: elastic

password: <PASSWORD_elastic>

Example 

URL: https://192.168.1.1

user: elastic

password: GeneratedPassword

# Disable repositories

```bash
sed -i "s/^deb/#deb/" /etc/apt/sources.list.d/wazuh.list
sed -i "s/^deb/#deb/" /etc/apt/sources.list.d/elastic-7.x.list
apt-get update
```

