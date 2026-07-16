# Day 4: Automating the routers with Ansible

## Why not SSH?

The routers don't run an SSH daemon — they were configured directly through
`docker exec ... vtysh`. Ansible normally manages hosts over SSH, but it also
ships a `docker` connection plugin (via the `community.docker` collection)
that runs modules through `docker exec` instead. Inventory hosts are
configured with `ansible_connection: docker`, and `ansible_host` is set to
the container name (e.g. `clab-noc-sim-r1`).

Ansible itself runs in its own container (no host install), given access to
the host's Docker socket so it can control the sibling router containers —
the same "control containers from a container" pattern used for the SNMP
sidecar in Day 2.

## Problem #1: `docker.io` doesn't include the Docker CLI

The Ansible container's image installs `apt install docker.io` to get the
`docker` client needed by the connection plugin. On Debian, this package
installs the Docker **daemon** (`dockerd`) and supporting services, but not
the client binary — `which docker` came back empty even though `dpkg -l`
showed the package as installed.

**Fix:** add Docker's official apt repository and install only
`docker-ce-cli` (the client package, no daemon):

```dockerfile
RUN apt-get install -y ca-certificates curl gnupg
RUN install -m 0755 -d /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
RUN echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian bookworm stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
```

## Config backup playbook

`backup-configs.yml` pulls `show running-config` from all three routers and
saves each one locally with a timestamp:

```yaml
- hosts: all
  tasks:
    - name: Get running config
      ansible.builtin.command: vtysh -c "show running-config"
      register: running_config

    - name: Save config locally
      delegate_to: localhost
      ansible.builtin.copy:
        content: "{{ running_config.stdout }}"
        dest: "backups/{{ inventory_hostname }}-{{ lookup('pipe', 'date +%Y%m%d-%H%M%S') }}.conf"
```

`delegate_to: localhost` on the save step is the key detail: the loop runs
per-router, but the *write* happens on the Ansible controller, not on the
router itself.

Run:
```
ansible-playbook -i inventory.yml backup-configs.yml
```

Result — real, timestamped config pulled straight from the router:
```
$ cat backups/r2-20260716-082219.conf

Building configuration...
Current configuration:
!
frr version 8.4_git
hostname r2
!
interface eth1
 ip address 10.0.12.2/30
exit
!
interface eth2
 ip address 10.0.23.1/30
exit
!
router ospf
 network 2.2.2.2/32 area 0
 network 10.0.12.0/30 area 0
 network 10.0.23.0/30 area 0
```

## Problem #2: `write memory` failed with "Unknown command"

The first version of the change-management playbook chained `configure
terminal` → `ip route ...` → `write memory` in one `vtysh` call. It failed:

```
"stdout": "% Unknown command: write memory"
```

`write memory` has to be run from the top-level exec mode, not while still
inside `configure terminal` — the same rule hit (and fixed) back in Day 1.
Adding an explicit `exit` before `write memory` fixed it.

## Change-management playbook

`change-static-route.yml` adds a static route on `r1` and immediately
verifies the result — not just that the command ran, but that the route is
actually live:

```yaml
- hosts: r1
  tasks:
    - name: Add static route
      ansible.builtin.command: >
        vtysh -c "configure terminal"
        -c "ip route 192.168.100.0/24 10.0.12.2"
        -c "exit"
        -c "write memory"

    - name: Show updated routing table
      ansible.builtin.command: vtysh -c "show ip route static"
      register: route_check
```

Verified output:
```
S>* 192.168.100.0/24 [1/0] via 10.0.12.2, eth1, weight 1, 00:01:00
```

`S>*` = static route, installed in the FIB — confirmation the change is
actually active, not just that the command exited 0.
