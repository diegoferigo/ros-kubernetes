ARG from=ros:melodic-ros-core
FROM ${from}

RUN apt-get update &&\
    apt-get install -y \
        net-tools \
        iputils-ping \
        dnsutils \
        nano \
    &&\
    rm -rf /var/lib/apt/lists/*

COPY check-roscore.py /usr/local/bin/check-roscore.py
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
COPY entrypoint.sh /usr/local/bin/entrypoint.sh

RUN chmod 755 /usr/local/bin/check-roscore.py &&\
    chmod 755 /usr/local/bin/entrypoint.sh &&\
    chmod 755 /usr/local/bin/entrypoint.sh

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["bash"]

