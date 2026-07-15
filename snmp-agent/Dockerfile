FROM debian:bookworm-slim

RUN apt-get update \
    && apt-get install -y --no-install-recommends snmpd snmp \
    && rm -rf /var/lib/apt/lists/*

COPY snmpd.conf /etc/snmp/snmpd.conf

CMD ["snmpd", "-f", "-Lo"]
