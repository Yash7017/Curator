# Curator
Setting up Curator for deleting the logs time to time. We need some mechanism in place to stop the Elasticsearch indices from growing endlessly. To achieve this, one can occasionally go to Kibana and delete the indices using GUI there, Why not automate the things? Elastic has provided a tool to curate or manage your Elasticsearch indices called Curator. One specific use case of the curator is applying a retention policy to your indices, deleting the indices that are older than a certain threshold.

# Installing Curator

A curator can be installed in various ways, you can find it here. Let’s download the DEB package, using
```
sudo wget https://packages.elastic.co/curator/5/debian9/pool/main/e/elasticsearch-curator/elasticsearch-curator_5.5.4_amd64.deb
```

 After downloading, install it using the following command:
 ```
 sudo dpkg -i elasticsearch-curator_5.5.4_amd64.deb
 ```
 Curator will now be installed at /opt/elasticsearch-curator location.
 
 We will create a folder and create a curator.yml file. 
 ```
 mkdir curator
 ```
 ```
 sudo nano /curator/curator.yml
 ```
 Then put the following configuration in the curator.yml file. 
 ```
 client:
        hosts:
                - master server privateIp
        port: 9200
        url_prefix:
        use_ssl: False
        certificate:
        client_cert:
        client_key:
        ssl_no_validate: False
        http_auth: "elastic:slfdslnionwol34"
        timeout: 30
        master_only: False


logging:
        loglevel: INFO
        logfile:
        logformat: default
        blacklist: [ 'elasticsearch', 'urllib3']
 ```
 If you want to understand the other components working of curator.yml file. then go through the document. https://www.elastic.co/guide/en/elasticsearch/client/curator/current/configfile.html
 
 **Note: If you have x-pack enabled for the ELK then put your username and password for elastic in http_auth: and replace the master server privateIP with your system private ip where you are installing your Curator**
 
 Now create a delete_indices_time_base.yml file in the same folder to delete the logs older than 15 days. 
 
 ```
 sudo nano curator/delete_indices_time_base.yml
```
Write all of your rules here, you can delete the logs in many ways eg; based on the number of logs should be there, based on the date etc. 

*Here we are gonna write the rule to delete the logs that are older than 15 days. it will only delete the logs that are on elasticsearch storage.*

```
actions:
        1:
                action: delete_indices
                description:
                        Delete indices with %Y.%m.%d in the name where that date is older than 15 days

                options:
                        ignore_empty_list: True
                        timeout_override:
                        continue_if_exception: False
                        disable_action: False

                filters:
                        - filtertype: age
                          source: creation_date
                          direction: older
                          unit: days
                          unit_count: 15

```

Before we setup automation script for curator we will check with dry run. 
```
sudo curator --config /curator/curator.yml --dry-run /curator/delete_indices_time_base.yml
```
**Note: here you give your own path to curator.yml and delete_indices_time_base.yml from the home directory.**

If it worked properly then Now we will setup the cronjob for running automatically everyday. 

###### script

Create curator.sh file in /etc/cron.daily/ directory.
```
sudo nano /etc/cron.daily/curator.sh
```
Now in this file we will add our rules to this file. 
```
#!/bin/sh
/opt/elasticsearch-curator/curator --config /curator/curator.yml /curator/delete_indices_time_base.yml
sudo crontab -e
0 0 * * * /bin/bash /etc/cron.daily/curator.sh
```
**Setup the path for your curator.yml file and rules file.**
It will delete all the logs at 12 AM everyday by running curator.sh






