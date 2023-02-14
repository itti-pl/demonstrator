FROM confluentinc/cp-kafka-connect:6.1.9
RUN confluent-hub install --no-prompt confluentinc/kafka-connect-elasticsearch:14.0.5
COPY ./config/add_connector_config.sh /etc/confluent/docker/
COPY ./config/start.sh /etc/confluent/docker/
USER root
RUN chown appuser:appuser /etc/confluent/docker/start.sh
RUN chown appuser:appuser /etc/confluent/docker/add_connector_config.sh
RUN chmod +x /etc/confluent/docker/start.sh
RUN chmod +x /etc/confluent/docker/add_connector_config.sh
USER appuser
CMD ["/etc/confluent/docker/start.sh"]
