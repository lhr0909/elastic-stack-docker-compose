# BELK Stack Setup
## Beats (Filebeat)
[Filebeat](https://www.elastic.co/guide/en/beats/filebeat/current/index.html) is a lightweight log shipper built for the Elastic Stack. It is written in Golang, and it doesn’t take up much resources on the host machine while shipping logs to Logstash, and has back-pressure sensing in case Logstash is overloaded.

You can find the sample Filebeat configuration here, and the main sections for the configuration in Filebeat is `input_type`s in `prospectors`, and `output` section. For the sample BELK setup, we are going to set `output` to Logstash from Filebeat. There are some basic filtering options built-in for Filebeat, such as multiline support and extra metadata, so we can mutate the logs somewhat before shipping to Logstash. When there are more logs to gather from more machines (>5G per day), we would want to have message queue setup in the stack and the first stop for the logs would be in a message queue like Apache Kafka. You can find more about configuring Filebeat [here](https://www.elastic.co/guide/en/beats/filebeat/current/configuring-howto-filebeat.html).

There are also other beats available, and I think [Metricbeat](https://www.elastic.co/guide/en/beats/metricbeat/5.5/index.html) would also be a good one to deploy to production.

## Logstash
Logstash is the de facto log shipper for Elasticsearch. It has powerful filtering options for parsing logs into meaningful key-value pairs before putting into Elasticsearch for indexing. It is written in Ruby and running in JRuby.

The are two types of configurations for Logstash, one is the Logstash Settings, which controls the execution of Logstash. You can find it [here](https://www.elastic.co/guide/en/logstash/current/logstash-settings-file.html).

The configuration for Logstash has 3 major sections: `input`, `filter`s, and `output`. You can find the sample config here, and more about the configuration [here](https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html).

## Elasticsearch
Elasticsearch is a data store and search engine that does most of the heavy lifting throughout the stack. It is written in Java and uses Apache Lucene under the hood for search indexing. It provides REST interface for ingesting and interacting with the data.

The configuration for Elasticsearch is quite complex, but the default settings can serve most of the needs if the data size is not too large. When the scale gets bigger, I will provide more info and reference as we go. For now I will just give a sample configuration and reference manual [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html).

### Important Note for Configuring Elasticsearch in Production

There is a gotcha where in production systems, `vm.max_map_count` needs to set to 262144 in order to be able to start Elasticsearch Instance, since by default it is doing a memory bootstrap check, and requires to increase the limits for mmap counts. See more [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html) and [here (under `Running ES from the command line -> Production mode` section)](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)

## Kibana
Kibana is a log viewer web interface written in node.js. It provides UI to query Elasticsearch and gives powerful charting and dashboarding features for users to view the logs in different ways. 

To get it up and running, we only need to provide the address for Elasticsearch, and there will be step-by-step guide in the web UI to help us setup the viewing of the logs.

The search queries are mostly based on the search query APIs from Elasticsearch, putting some reference [here]().

## Docker Deployment
If there is already Docker setup, the docker-compose file I am providing should spin up BELK all in one machine, with configurable options via bind-mounting configs and environment variables. They are going to be connected inside Docker’s network, but I will point out the network address configurations in case we want to switch to bare metal deployment, and cluster deployment.

When you run `docker-compose up -d`, it will pull the official images from Elastic and load up the configurations into the container. Wait for about 3-5 minutes for everything to kick in, then you can visit Kibana at `localhost:5601`, and put in username `elastic`, password `changeme` to login, and at the startup page when it is prompted to configure an index pattern, use `filebeat-*` instead of `logstash-*`, and once you hit create, you can go back to the discover tab and see the logs show up on Kibana. Default logs are in date range July 26, 2017 to July 28, 2017.

The data directory is managed via [named volumes](https://docs.docker.com/engine/admin/volumes/volumes/). When `docker-compose up` is called, it will create the corresponding volumes in Docker and you can find them using `docker volume ls` and `docker volume inspect <volume_name>` to see the mountpoint on the host machine. Normally when we tear down the containers using `docker-compose down`, the volumes will be retained, only when we use `docker-compose down -v`, that's when the volumes will also be torn down. More about this [here](https://docs.docker.com/compose/reference/down/)