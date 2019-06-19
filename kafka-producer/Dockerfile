FROM wurstmeister/kafka
COPY ./quotes.json /tmp
CMD /opt/kafka/bin/kafka-producer-perf-test.sh --num-records 10000 --throughput 1 --topic test --payload-file /tmp/quotes.json --producer-props bootstrap.servers=kafka:9092
