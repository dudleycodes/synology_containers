# Synology Container Projects

## Assumptions

* At least one Synology NIC is bound to `172.30.1.1`.
* This git repo is cloned to shared folder used by Synology's 'container manager'.

## Prerequisites

### MACVLAN Network Driver

Use SSH to create a docker network named `eth5_macvlan`, using the `macvlan` driver, and attaching to the default
network interface (eth5). This provides a range of IP address containers can use to directly attach to the network.

Range: 172.30.1.33 - 172.30.1.62

```shell
sudo docker network create -d macvlan -o parent=eth5 --subnet=172.30.0.0/18 --gateway=172.30.0.1 --ip-range=172.30.1.32/27 eth5_macvlan
```

## Projects

| Name                                          | Binding
| :-------------------------------------------- |
| dns                                           |
| └─── adgaurd                                  |
| [Grafana](./grafana/)                         |
| ├─── grafana                                  |
| └─── proxy                                    | 172.30.1.34
| [heimdall](./heimdall/)                       | 172.30.1.33
| [kiwix](./kiwix/)                             |
| ├─── kiwix-serve                              |
| └─── proxy                                    | 172.30.1.35
| [monitoring-synology](./monitoring-synology/) |
| ├─── cadvisor                                 |
| ├─── node_exporter                            |
| ├─── prometheus                               |
| └─── proxy                                    |
| [monitoring-unifi](./monitoring-unifi/)       |
| ├─── prometheus                               | :5011/prometheus/
| ├─── unpoller                                 | :5011/unpoller/
| └─── proxy                                    | :5011/
| [pxe-server](./pxe-server/)                   |
| └─── netbootxyz                               | 172.30.1.36

## General Practices

* Healthchecks should be configured for all services.
* To make a service available to the network with a DNS entry, it is attached to a preconfigured `macvlan` network.
* If a project runs on the host network or maps multiple ports to the host, for HTTP(s) traffic, use a reverse proxy, to
    expose services one using one port with sub paths.

### Damage Reduction

The more locked-down a container can be from access filesystem or network resources, the better.

* If a container doesn't need DNS support, disable it with `dns: 0.0.0.0`.
* If the containers on a network don't need to communicate with outside networks, disable by setting
    `driver_opts: com.docker.network.bridge.enable_ip_masquerade: 'false'`.
* using read only volume mounts
