FROM registry.cn-hangzhou.aliyuncs.com/flink-streaming-platform-web/flink-web:flink-1.13.2

RUN apt-get update; \
apt-get install -y vim --fix-missing;

# Copy configuration
COPY conf/* /data/projects/flink-streaming-platform-web/conf/

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
