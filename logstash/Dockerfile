#ARG ELK_VERSION

# https://github.com/elastic/logstash-docker
#FROM docker.elastic.co/logstash/logstash-oss:${ELK_VERSION}

FROM docker.elastic.co/logstash/logstash:6.7.1

# Add your logstash plugins setup here
# Example: RUN logstash-plugin install logstash-filter-json

RUN logstash-plugin install logstash-filter-age
RUN logstash-plugin install logstash-output-slack
