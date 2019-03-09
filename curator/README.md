# Curator

Elasticsearch Curator helps you curate or manage your indices.

## Contents

1. [Installation](#installation)
   * [Host setup](#host-setup)
2. [Configuration](#configuration)
   * [Configure Curator](#configure-curator)
3. [Dry Run](#dry-run)
4. [Delete old indices](#delete-old-indices)

## Installation

### Host setup
Install Curator on Linux using pip command.
```console
$ sudo apt-get install python-pip

Or

$ sudo yum install python-pip

$ sudo pip install Elasticsearch-curator
```

Verify Curator installed properly:
```console
$ which curator
/usr/local/bin/curator

$ /usr/local/bin/curator --version
```
A good practice is to install on Elasticsearch machine itself.

If your log size is more and you want to keep old data for 5days as per your requirement then you need to delete old Elasticsearch indices where all logs get stored and these results free up some disk space for newly generated logs. And you will be knowing Logstash create new index every day this is default configuration.


## Configuration

### Configure Curator

Curator configuration file `curator.yml`
```console
---
# Remember, leave a key empty if there is no value. None will be a string,
# # not a Python "NoneType"
client:
  hosts:
    - 127.0.0.1
  port: 9200
  url_prefix:
  use_ssl: False
  certificate:
  client_cert:
  client_key:
  aws_key:
  aws_secret_key:
  aws_region:
  ssl_no_validate: False
  http_auth:
  timeout: 30
  master_only: False

logging:
  loglevel: INFO
  logfile: "/var/log/curator/curator.log"
  logformat: default
```

Curator action file `delete-indices.yml`
```console
actions:
  1:
    action: delete_indices
    description: >-
      Delete indices older than 5 days (based on index name), for logstash-
      prefixed indices. Ignore the error if the filter does not result in an
      actionable list of indices (ignore_empty_list) and exit cleanly.
    options:
      ignore_empty_list: True
      timeout_override:
      continue_if_exception: False
      disable_action: False
    filters:
    - filtertype: pattern
      kind: prefix
      value: prod-
      exclude:
    - filtertype: age
      source: name
      direction: older
      timestring: '%Y.%m.%d'
      unit: days
      unit_count: 5
      exclude:
  2:
    action: delete_indices
    description: >-
      Delete indices older than 3 days (based on index name), for logstash-
      prefixed indices. Ignore the error if the filter does not result in an
      actionable list of indices (ignore_empty_list) and exit cleanly.
    options:
      ignore_empty_list: True
      timeout_override:
      continue_if_exception: False
      disable_action: False
    filters:
    - filtertype: pattern
      kind: prefix
      value: dev-
      exclude:
    - filtertype: age
      source: name
      direction: older
      timestring: '%Y.%m.%d'
      unit: days
      unit_count: 3
      exclude:
  3:
    action: delete_indices
    description: >-
      Delete indices in excess of 65 gigabytes
    options:
      ignore_empty_list: true
    filters:
    - filtertype: pattern
      kind: prefix
      value: dev-
    - filtertype: space
      disk-space: 65
      use_age: true
      source: creation_date
```

## Dry Run

Now, Goto the location where you have created “delete-indices.yml” action file and check which all indices are going to delete with dry-run option. Dry-run option is used to test action file it will not delete the index.

```console
$ sudo curator ./delete-indices.yml --config ./curator.yml --dry-run
```

## Delete old indices
To cleanup old indices run below command
```console
$ sudo curator ./delete-indices.yml --config ./curator.yml
```

## Documentation

https://github.com/elastic/curator
