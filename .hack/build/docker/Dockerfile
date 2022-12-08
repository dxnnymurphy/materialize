FROM ubuntu:22.04

RUN apt update
RUN apt install -y postgresql-client

RUN mkdir -p /scripts
ADD config/myscript /scripts
WORKDIR /scripts
RUN chmod +x myscript
CMD ["/scripts/materialize-provisioning", "--run", "materialized"]