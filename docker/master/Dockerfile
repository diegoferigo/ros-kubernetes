ARG from=ros:melodic-ros-core
FROM ${from}

RUN apt-get update &&\
    apt-get install -y \
        net-tools \
        iputils-ping \
        dnsutils \
    &&\
    rm -rf /var/lib/apt/lists/*

COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod 755 /usr/local/bin/entrypoint.sh

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["roscore"]
