# Filebeat Installation
[Getting Started With Filebeat](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-getting-started.html#filebeat-getting-started)



### Filebeat Container

On distributions which have SELinux enabled out-of-the-box you will need to either re-context the files or set SELinux
into Permissive mode in order for docker-elk to start properly. For example on Redhat and CentOS, the following will
apply the proper context:

```console
$ docker run -d --name=filebeat --user=root --volume="/home/ec2-user/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro" --volume="/opt/logs/:/opt/logs/:ro" --volume="/opt/logstash/data:/usr/share/filebeat/data:rw" --volume="/opt/logstash/logs:/usr/share/filebeat/logs:rw" --log-driver json-file --log-opt max-size=10m --log-opt max-file=10 --hostname prod-node-1 docker.elastic.co/beats/filebeat:6.6.1
```
