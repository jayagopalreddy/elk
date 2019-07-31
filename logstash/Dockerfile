#ARG ELK_VERSION

# https://github.com/elastic/logstash-docker
#FROM docker.elastic.co/logstash/logstash-oss:${ELK_VERSION}

FROM docker.elastic.co/logstash/logstash:6.8.1

# Add your logstash plugins setup here
# Example: RUN logstash-plugin install logstash-filter-json

RUN logstash-plugin install logstash-filter-age
RUN logstash-plugin install logstash-input-cloudwatch_logs
RUN logstash-plugin install logstash-codec-cloudtrail
RUN logstash-plugin install logstash-filter-json
RUN logstash-plugin install logstash-filter-split
RUN logstash-plugin install logstash-output-slack
