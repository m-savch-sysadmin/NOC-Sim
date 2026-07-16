FROM python:3.12-slim

RUN apt-get update
RUN apt-get install -y ca-certificates
RUN apt-get install -y curl
RUN apt-get install -y gnupg

RUN install -m 0755 -d /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
RUN chmod a+r /etc/apt/keyrings/docker.asc

RUN echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian bookworm stable" > /etc/apt/sources.list.d/docker.list

RUN apt-get update
RUN apt-get install -y docker-ce-cli

RUN pip install ansible
RUN pip install docker

WORKDIR /ansible
