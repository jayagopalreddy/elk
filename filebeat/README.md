# Filebeat Installation
[Getting Started With Filebeat](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-getting-started.html#filebeat-getting-started)



### Filebeat Container

The data stored in Logstash will be persisted after container reboot but not after container removal.

In order to persist Logstash data even after removing the Logstash container, you'll have to mount a volume on
your Docker host. Update the `logstash` service declaration to:

```yml
logstash:

  volumes:
    - /data/logstash:/usr/share/logstash/data/
```
