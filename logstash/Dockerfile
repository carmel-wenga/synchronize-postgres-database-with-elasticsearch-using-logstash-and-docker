FROM docker.elastic.co/logstash/logstash:8.7.0

# copy the postgres jdbc driver to the external library folder of logstash
COPY ./bin/ /usr/share/logstash/logstash-core/lib/jars/

COPY config/ /usr/share/logstash/config/

CMD ["logstash", "-f", "/usr/share/logstash/config/usersync.conf"]


