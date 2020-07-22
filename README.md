# elk stack example
 
 ### Run
 
 ```
  docker-compose up -d
 ```   
 
In this example , we feed generated log files to directly to logstash container. 

Server Code generate log files and send to logstash

```xml

   <appender name="logstash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <param name="Encoding" value="UTF-8"/>
        <remoteHost>logstash</remoteHost>
        <port>5000</port>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <customFields>{"app_name":"elk-search", "app_port": "8080"}</customFields>
        </encoder>
   </appender>

```
In logstash.conf  we can get logs from these location and pass to elasticsearch

```
input {
    udp {
        port => "5000"
        type => syslog
        codec => json
    }
    tcp {
        port => "5000"
        type => syslog
        codec => json_lines
    }
    http {
        port => "5001"
        codec => "json"
    }
}

filter {
    if [type] == "syslog" {
        mutate {
            add_field => { "instance_name" => "%{app_name}-%{host}:%{app_port}" }
        }
    }
}
 
output {
    elasticsearch {
        hosts => [ "elasticsearch:9200" ]
        index => "filelogs-%{+YYYY.MM.dd}"
    }
     stdout {
        codec => rubydebug
     }
}

```
 
