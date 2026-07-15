# NOC-Sim

Home-lab sieciowy: symulowana topologia 3 routerów z OSPF, monitorowana przez
pełny stos SNMP → Prometheus → Grafana. Zbudowane, żeby przećwiczyć routing,
troubleshooting i observability tak, jak realnie wygląda to w pracy NOC/SRE.

W budowie — to jest wersja robocza. Pełne README (z diagramem architektury,
instrukcją odtworzenia od zera i opisem napotkanych problemów) w toku.

## Co mam

- **`topology.clab.yml`** — topologia Containerlab: 3 routery FRR (r1-r2-r3)
  połączone w linię, OSPF area 0.
- **`snmp-agent/`** — kontener-sidecar (Debian + net-snmp) dostarczający SNMP
  dla routerów — obraz FRR na Alpine miał niesprawny moduł interfejsów.
- **`prometheus/`** — konfiguracja scrape'owania SNMP przez `snmp_exporter`.
- **`grafana/`** — automatycznie provisionowane źródło danych i dashboard
  (`network-overview.json`) pokazujący stan i ruch na interfejsach.

## Stack

Containerlab · FRRouting (OSPF) · Docker · net-snmp · snmp_exporter ·
Prometheus · Grafana

## Status

- [x] Topologia i OSPF (pełne sąsiedztwo, trasy nauczone automatycznie)
- [x] SNMP monitoring wszystkich routerów
- [x] Prometheus scrape'uje metryki interfejsów
- [x] Dashboard Grafana z podglądem stanu i ruchu na żywo
- [ ] Automatyzacja Ansible (backup configów)
- [ ] Eskalacja alertów (Alertmanager → ticket → Telegram/email)
- [ ] Symulacja awarii + raport incydentu
- [ ] Pełne README z diagramem i instrukcją uruchomienia od zera
