FROM registry.cn-hangzhou.aliyuncs.com/flink-streaming-platform-web/flink-web:flink-1.13.2

# Copy configuration
COPY conf/* /data/projects/flink-streaming-platform-web/conf/

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
