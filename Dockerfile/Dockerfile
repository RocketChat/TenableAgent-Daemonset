FROM bitnami/minideb:latest

ARG NESSUS_SHA256="3e3287344dd133db0402c9ea313b75ce5de86b7128292d22c427ff57f5a5bc61  NessusAgent-10.3.1-ubuntu1404_amd64.deb"
ARG NESSUS_URL="https://d2530cnixqlh3k.cloudfront.net/nessus-agent/NessusAgent-10.3.1-ubuntu1404_amd64.deb"

LABEL maintainer="security@rocket.chat"

#Download updates and Nessus Agent
RUN apt update -y && apt install -y --no-install-recommends gettext-base curl 
RUN curl -s -k $NESSUS_URL -o NessusAgent-install.deb

#Check if the downloaded file have the same SHA256 hash provided by tenable in https://www.tenable.com/downloads/nessus-agents?loginAttempted=true
#Aborts if failed

RUN echo $NESSUS_SHA256 > sha256_signature.txt
RUN if [ $(sha256sum -c sha256_signature.txt | awk -F ": " '{print $2}') != "OK" ]; then exit 1; else echo "Checksum OK"; fi;

#Install Nessus

RUN dpkg -i NessusAgent-install.deb && rm -rf NessusAgent-install.deb
