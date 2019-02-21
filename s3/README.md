This document describes configuring a logstash parser to read CSV data from an s3 bucket,
parse the data and output to elasticsearch. It also provides a working example of how to
parse longitude and latitude fields into a geo_point field in Elasticsearch.

For this proof of concept, an ELK (Elasticsearch, Logstash, Kibana) stack was set up on an
AWS EC2 instance. An s3 bucket was also configured in the same AWS environment and the relevant
CSV file was uploaded to it. 

For the EC2 to access the s3 bucket, it was assigned an IAM role which granted access. 
The EC2 instance can also make use of AWS access keys to read/write from the s3 bucket,
but the role assignment is the preferred method as it is more secure.

Below is the logstash configuration used;

    input{
            s3 {
                    "bucket" => "example-s3"
                    "prefix" => "airport"
            }
    }

    filter {
            csv {
                    skip_header => "true"
                    columns => ["id", "ident", "type", "name", "latitude_deg", "longitude_deg","elevation_ft","continent","iso_country","iso_region","municipality","scheduled_service","gps_code","iata_code","local_code","home_link","wikipedia_link","keywords"]
            }

            mutate {
                    convert => ["id", "integer"]
                    convert => ["latitude_deg", "float"]
                    convert => ["longitude_deg", "float"]
                    convert => ["elevation_ft", "integer"]

                    add_field => { "[location][lat]" => "%{latitude_deg}" }
                    add_field => { "[location][lon]" => "%{longitude_deg}" }
            }

    }

    output {
            elasticsearch {
                    hosts => ["127.0.0.1"]
                    index => "airport"
            }
    }

*Note-1:* The IP address specified in the elasticsearch output should point to the 
internal IP of the EC2 instance where the ELK stack is set up. 

*Note-2:* Elasticsearch and Kibana should be started before Logstash and the 
following commands run in the dev tools section of the Kibana environment.

Create the output index

    PUT airport

Create the mapping for the geo_point property in the output index.

    PUT airport/_mapping/doc
    {
        "properties": {
            "location": {
                "type": "geo_point"
            }
        }
    }

Running these commands before satrting the logstash pipeline is required for 
the geo_point to be correctly indexed into the sample airport index. 