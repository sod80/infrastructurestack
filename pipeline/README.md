The following document describes the changes made to the infrastructure project.

## Multiple Logstash Pipelines

The docker image used for this proof of concept is "sebp/elk". This image when run in docker will create an entire
ELK stack within a container. The Logstash within the container reads from a locally running MySQL database 
instance and indexes documents to the elasticsearch within the conatainer and an external elasticsearch. 
To override the default values for the logstash component within the container, Docker volumes are used to mount 
the required configuration files and MySQL connector JAR to the docker container.

The following command was executed within the project directory to create the docker container;
    
    docker run -v $PWD/config:/etc/logstash/conf.d/ -v $PWD/pipeline/pipelines.yml:/opt/logstash/config/pipelines.yml -v $PWD/data/mysql-connector-java-5.1.47-bin.jar:/home/mysql-connector-java-5.1.47-bin.jar -p 5601:5601 -p 9200:9200 -p 5044:5044 -it sebp/elk


## ID based last run configuration

Additional changes where made to the input filter in the logstash configuration files. These changes 
require logstash to operate with an id/numeric based last run policy as opposed to timestamp based. 

The following configurations were added to the input filter in the logstash configuration file/s;

    use_column_value => true
    tracking_column => "id"
    tracking_column_type => "numeric"

*Note:* These values must be declared before the statement field in the input filter. 