# Filebeat Installation
[Getting Started With Filebeat](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-getting-started.html#filebeat-getting-started)



### Filebeat Container

Run filebeat container by using below command:

```console
$ docker run -d --name=filebeat --user=root --volume="/home/ec2-user/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro" --volume="/opt/logs/:/opt/logs/:ro" --volume="/opt/logstash/data:/usr/share/filebeat/data:rw" --volume="/opt/logstash/logs:/usr/share/filebeat/logs:rw" --log-driver json-file --log-opt max-size=10m --log-opt max-file=10 --hostname test-node-1 docker.elastic.co/beats/filebeat:6.6.1
```

We need to mount our logs here (which we need to shift to logstash) `--volume="/opt/logs/:/opt/logs/:ro"`

### Configuring Filebeat

Filebeat configuration file `sudo vim /home/ec2-user/filebeat.yml`

```console
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /opt/logs/prod.log
  fields:
    type: qfchat
    environment: PROD
    layer: App
  multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
  multiline.negate: true
  multiline.match: after  
output.logstash:
  hosts: ["172.0.0.07:5044"]
```
