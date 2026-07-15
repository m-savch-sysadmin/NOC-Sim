# NOC-Sim

A network home-lab: a simulated 3-router OSPF topology, monitored through a
full SNMP → Prometheus → Grafana stack. Built to practice routing,
troubleshooting, and observability the way they actually show up in NOC/SRE
work.

> 🚧 Work in progress — this is a draft README. A full version (architecture
> diagram, from-scratch setup instructions, and a write-up of the problems
> hit along the way) is coming.

## What's in here

- **`topology.clab.yml`** — Containerlab topology: 3 FRR routers (r1-r2-r3)
  in a line, OSPF area 0.
- **`snmp-agent/`** — a sidecar container (Debian + net-snmp) providing SNMP
  for the routers — the FRR image on Alpine had a broken interface module.
- **`prometheus/`** — scrape configuration for SNMP via `snmp_exporter`.
- **`grafana/`** — auto-provisioned datasource and dashboard
  (`network-overview.json`) showing interface status and traffic.

## Stack

Containerlab · FRRouting (OSPF) · Docker · net-snmp · snmp_exporter ·
Prometheus · Grafana

## Status

- [x] Topology and OSPF (full adjacency, routes learned automatically)
- [x] SNMP monitoring on all routers
- [x] Prometheus scraping interface metrics
- [x] Grafana dashboard with live status and traffic
- [ ] Ansible automation (config backup)
- [ ] Alert escalation (Alertmanager → ticket → Telegram/email)
- [ ] Simulated outage + incident report
- [ ] Full README with architecture diagram and from-scratch setup guide
