# Elastic stack (ELK) on Docker


Elasticsearch, Logstash, and Kibana (formerly known as the [ELK Stack](https://www.elastic.co/elk-stack), now renamed to the Elastic Stack with the addition of Beats) deliver actionable insights in real time for almost any type of structured and unstructured data source, at any scale. From centralizing infrastructure logs, traces, and metrics to deliver a great end user experience to keeping a pulse on your fleet of smart devices using real-time telemetry - the possibilities are endless with the Elastic Stack.

Based on the official Docker images from Elastic:

* [elasticsearch](https://www.docker.elastic.co/#)
* [logstash](https://www.docker.elastic.co/#)
* [kibana](https://www.docker.elastic.co/#)


## Contents

1. [Requirements](#requirements)
   * [Host setup](#host-setup)
   * [SELinux](#selinux)
2. [Configuration](#configuration)
   * [How can I tune the Kibana configuration?](#how-can-i-tune-the-kibana-configuration)
   * [How can I tune the Logstash configuration?](#how-can-i-tune-the-logstash-configuration)
   * [How can I tune the Elasticsearch configuration?](#how-can-i-tune-the-elasticsearch-configuration)
3. [Storage](#storage)
   * [How can I persist Elasticsearch data?](#how-can-i-persist-elasticsearch-data)
   * [How can I persist Logstash data?](#how-can-i-persist-logstash-data)
4. [JVM tuning](#jvm-tuning)
   * [How can I specify the amount of memory used by a service?](#how-can-i-specify-the-amount-of-memory-used-by-a-service)
5. [Usage](#usage)
   * [Bringing up the stack](#bringing-up-the-stack)
   * [Initial setup](#initial-setup)
6. [Sample setup](#sample-setup)
   * [Launch an AWS EC2 Instance](#Launch-an-AWS-EC2-Instance)
   * [Install required packages and necessary changes](#Install-required-packages-and-necessary-changes)
   * [Docker Stack](#docker-stack)

## Requirements

### Host setup

1. Install [Docker](https://docs.docker.com/install/)
2. Install [Docker Swarm](https://docs.docker.com/engine/swarm/swarm-mode/)
3. Clone this repository

### SELinux

On distributions which have SELinux enabled out-of-the-box you will need to either re-context the files or set SELinux
into Permissive mode in order for docker-elk to start properly. For example on Redhat and CentOS, the following will
apply the proper context:

```console
$ chcon -R system_u:object_r:admin_home_t:s0 elk/
```

## Configuration

**NOTE**: Configuration is not dynamically reloaded, you will need to restart the stack after any change in the
configuration of a component.

### How can I tune the Kibana configuration?

The Kibana default configuration is stored in `kibana/config/kibana.yml`.

It is also possible to map the entire `config` directory instead of a single file.

### How can I tune the Logstash configuration?

The Logstash configuration is stored in `logstash/config/logstash.yml`.

It is also possible to map the entire `config` directory instead of a single file, however you must be aware that
Logstash will be expecting a
[`log4j2.properties`](https://github.com/elastic/logstash-docker/tree/master/build/logstash/config) file for its own
logging.

### How can I tune the Elasticsearch configuration?

The Elasticsearch configuration is stored in `elasticsearch/config/elasticsearch.yml`.

You can also specify the options you want to override directly via environment variables:

```yml
elasticsearch:

  environment:
    network.host: "_non_loopback_"
    cluster.name: "my-cluster"
```


## Storage

### How can I persist Elasticsearch data?

The data stored in Elasticsearch will be persisted after container reboot but not after container removal.

In order to persist Elasticsearch data even after removing the Elasticsearch container, you'll have to mount a volume on
your Docker host. Update the `elasticsearch` service declaration to:

```yml
elasticsearch:

  volumes:
    - /data/elasticsearch:/usr/share/elasticsearch/data
```

This will store Elasticsearch data inside `/data/elasticsearch`.

### How can I persist Logstash data?

The data stored in Logstash will be persisted after container reboot but not after container removal.

In order to persist Logstash data even after removing the Logstash container, you'll have to mount a volume on
your Docker host. Update the `logstash` service declaration to:

```yml
logstash:

  volumes:
    - /data/logstash:/usr/share/logstash/data/
```

This will store Logstash data inside `/data/logstash`.

**NOTE:** beware of these OS-specific considerations:
* **Linux:** the [unprivileged `elasticsearch` user][esuser] is used within the Elasticsearch image, therefore the
  mounted data directory must be owned by the uid `1000`.
* **macOS:** the default Docker for Mac configuration allows mounting files from `/Users/`, `/Volumes/`, `/private/`,
  and `/tmp` exclusively. Follow the instructions from the [documentation][macmounts] to add more locations.

[esuser]: https://github.com/elastic/elasticsearch-docker/blob/016bcc9db1dd97ecd0ff60c1290e7fa9142f8ddd/templates/Dockerfile.j2#L22
[macmounts]: https://docs.docker.com/docker-for-mac/osxfs/


## JVM tuning

### How can I specify the amount of memory used by a service?

By default, both Elasticsearch and Logstash start with [1/4 of the total host
memory](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/parallel.html#default_heap_size) allocated to
the JVM Heap Size.

The startup scripts for Elasticsearch and Logstash can append extra JVM options from the value of an environment
variable, allowing the user to adjust the amount of memory that can be used by each component:

| Service       | Environment variable |
|---------------|----------------------|
| Elasticsearch | ES_JAVA_OPTS         |
| Logstash      | LS_JAVA_OPTS         |

To accomodate environments where memory is scarce (Docker for Mac has only 2 GB available by default), the Heap Size
allocation is capped to 1GB per Logstash service and 2GB per elasticsearch service in the `docker-stack.yml` file. If you want to override the
default JVM configuration, edit the matching environment variable(s) in the `docker-stack.yml` file.

For example, to increase the maximum JVM Heap Size for Logstash:

```yml
logstash:

  environment:
    LS_JAVA_OPTS: "-Xmx2g -Xms2g"
```

## Usage

### Bringing up the stack

**Note**: In case you updated a base image - you may need to run `docker stack deploy` first

Start the stack using `docker stack deploy`:

```console
$ docker stack deploy -c docker-stack.yml elk
```

Give Kibana a few seconds to initialize, then access the Kibana web UI by hitting
[http://localhost:5601](http://localhost:5601) with a web browser.

By default, the stack exposes the following ports:
* 5000: Logstash TCP input
* 5044: Logstash Filebeat input
* 9600: Logstash API
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP transport
* 5601: Kibana UI

## Initial setup

### Default Kibana index pattern creation

When Kibana launches for the first time, it is not configured with any index pattern.

#### Via the Kibana web UI

**NOTE**: You need to inject data into Logstash before being able to configure a Logstash index pattern via the Kibana web
UI. Then all you have to do is hit the *Create* button.

Refer to [Connect Kibana with
Elasticsearch](https://www.elastic.co/guide/en/kibana/current/connect-to-elasticsearch.html) for detailed instructions
about the index pattern configuration.

#### On the command line

Create an index pattern via the Kibana API:

```console
$ curl -XPOST -D- 'http://localhost:5601/api/saved_objects/index-pattern' \
    -H 'Content-Type: application/json' \
    -H 'kbn-version: 6.6.0' \
    -d '{"attributes":{"title":"prod-*","timeFieldName":"@timestamp"}}'
```

The created pattern will automatically be marked as the default index pattern as soon as the Kibana UI is opened for the first time.

## Sample setup

### Launch an AWS EC2 Instance

[Describes the steps of launching a new Linux EC2 instance](https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-1-launch-instance.html)

### Install required packages and necessary changes

I am using Amazon Linux with the instance type t3.medium (2 vCPUs, 4GB Memory).

Install Git and clone the repo:

```console
$ sudo yum install git -y
$ git clone https://github.com/jayagopalreddy/elk.git
$ cd elk/
```

Update the packages and install Docker and Docker Swarm:

```console
$ sudo yum update -y
$ sudo yum install docker -y
$ sudo service docker start
$ sudo docker swarm init
$ sudo usermod -a -G docker ec2-user
```
You should then be able to run all of the docker commands without requiring sudo. After running the last command, need to logout and log back in for the change to take effect.



Create directories to persist Elasticsearch and Logstash data even after removing the containers

```console
$ mkdir -p /data/elasticsearch
$ mkdir -p /data/logstash
$ sudo chown -R 1000:1000 /data/
```


Increase open file limits of docker daemon in this path `/etc/sysconfig/docker`, and paste the below content.
```console
# The max number of open files for the daemon itself, and all
# running containers.  The default value of 1048576 mirrors the value
# used by the systemd service unit.
DAEMON_MAXFILES=1048576

# Additional startup options for the Docker daemon, for example:
# OPTIONS="--ip-forward=true --iptables=true"
# By default we limit the number of open files per container
OPTIONS="--default-ulimit nofile=65536:65536"

# How many seconds the sysvinit script waits for the pidfile to appear
# when starting the daemon.
DAEMON_PIDFILE_TIMEOUT=10
```


Restart the Docker service.
```console
$ sudo service docker restart
```


Apply kernel setting on host(optional)
```console
$ sudo sysctl -w vm.max_map_count=262144
```

### Docker Stack

 Deployed the swarm cluster using the following command:

```console
$ docker stack deploy -c docker-stack.yml elk
```

If all components get deployed without any error, the following command will show 3 running services:

```console
$ docker stack services elk
```

To check the service logs by using below commands:

```console
$ docker service logs -f elk_elasticsearch
$ docker service logs -f elk_logstash
$ docker service logs -f elk_kibana
```

Get the list of indices:
```console
$ curl "localhost:9200/_cat/indices?pretty&s=i"
```

Remove the Docker stack:

```console
$ docker stack rm elk
```
