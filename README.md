## Pre-req

A limit on mmap counts equal to 262,144 or more

!! This is the most frequent reason for Elasticsearch failing to start since Elasticsearch version 5 was released.

On Linux, use sysctl vm.max_map_count on the host to view the current value, and see Elasticsearch's documentation on virtual memory for guidance on how to change this value. Note that the limits must be changed on the host; they cannot be changed from within a container.

If using Docker for Mac, then you will need to start the container with the MAX_MAP_COUNT environment variable (see Overriding start-up variables) set to at least 262144 (using e.g. docker's -e option) to make Elasticsearch set the limits on mmap counts at start-up time.

You can do that in 2 ways.

Temporary set max_map_count:

    sudo sysctl -w vm.max_map_count=262144 

but this will only last till you restart your system.

Permament
In your host machine

    vi /etc/sysctl.conf

make entry vm.max_map_count=262144 and restart

## Quick Commands


### Start using Docker compose

    $ docker-compose up elk

### Start using Docker

    $ docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it --name elk sebp/elk

## Modifying config files:

Logstash's settings are defined by the configuration files 

    logstash.yml, jvm.options, pipelines.yml

located in 

    /opt/logstash/config.

Pipeline files are located in 

    /etc/logstash/conf.d  (e.g. 01-lumberjack-input.conf, 02-beats-input.conf) 

To load the logstash override files

    $ sudo docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it --name elk sebp/elk \
        -v ./logstash/30-output.conf:/etc/logstash/conf.d/30-output.conf \

or mount the whole dir

    $ sudo docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it --name elk sebp/elk \
        -v ./logstash/:/etc/logstash/conf.d/ \


docker run -v $PWD/logstash:/etc/logstash/conf.d -p 5601:5601 -p 9200:9200 -p 5044:5044 -it sebp/elk