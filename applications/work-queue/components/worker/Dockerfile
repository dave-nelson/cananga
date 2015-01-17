# Image: queue worker

FROM ubuntu:14.10

RUN apt-get -y update && apt-get -y upgrade

# Prerequisits
RUN apt-get -y install amqp-tools curl

# Copy scripts and utilities
COPY scripts /usr/local/bin
