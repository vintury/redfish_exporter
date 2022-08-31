# redfish_exporter
A prometheus exporter to get  metrics from redfish based servers such as lenovo/dell/Supermicro servers.

## Configuration

An example configure given as an [example][1]:
```yaml
hosts:
  10.36.48.24:
    username: admin
    password: pass
  default:
    username: admin
    password: pass
groups:
  group1:
    username: group1_user
    password: group1_pass
```
Note that the ```default``` entry is useful as it avoids an error
condition that is discussed in [this issue][2].

## Building

To build the redfish_exporter executable run the command:
```sh
make build
```

or build in centos 7 docker image
```sh
make docker-build-centos7
```

or build in centos 8 docker image
```sh
make docker-build-centos8
```
or we can also build a docker image  using [Dockerfile](./Dockerfile)

## Running
- running directly on linux
  ```sh
  redfish_exporter --config.file=redfish_exporter.yml
  ```
  and run   `redfish_exporter -h
  `  for more options.

- running in container
  
  Also if you build it as a docker image, you can also run in container, just remember to replace your config  `/etc/prometheus/redfish_exporter.yml` in container
## Scraping

We can get the metrics via
```
curl http://<redfish_exporter host>:9610/redfish?target=10.36.48.24

```
or by pointing your favourite browser at this URL.

## Reloading Configuration
```
PUT /-/reload
POST /-/reload
```
The `/-/reload` endpoint triggers a reload of the redfish_exporter configuration.
500 will be returned when the reload fails.

Alternatively, a configuration reload can be triggered by sending `SIGHUP` to the redfish_exporter process as well.

## Prometheus Configuration

You can then setup [Prometheus][3] to scrape the target using
something like this in your Prometheus configuration files:
```yaml
  - job_name: 'redfish-exporter'

    # metrics_path defaults to '/metrics'
    metrics_path: /redfish

    # scheme defaults to 'http'.

    static_configs:
    - targets:
       - 10.36.48.24 ## here is the list of the redfish targets which will be monitored
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9610  ### the address of the redfish-exporter address, hence relpace localhost with the server IP address that redfish-export is running on
      # (optional) when using group config add this to have group=my_group_name
      - target_label: __param_group
        replacement: my_group_name
```
Note that port 9610 has been [reserved][4] for the redfish_exporter.
## Supported Devices (tested)
- Enginetech EG520R-G20 (Supermicro Firmware Revision 1.76.39)
- Enginetech EG920A-G20 (Huawei iBMC 6.22)
- Lenovo ThinkSystem SR850 (BMC 2.1/2.42)
- Lenovo ThinkSystem SR650 (BMC 2.50)
- Dell PowerEdge R440, R640, R650, R6515, C6420
- GIGABYTE G292-Z20, G292-Z40, G482-Z54

## Grafana dashboard
<img width="1737" alt="Снимок экрана 2022-08-31 в 15 01 02" src="https://user-images.githubusercontent.com/4948177/187674246-11b56252-3ffb-40bf-8b51-d9acfd1573a9.png">
<img width="1722" alt="Снимок экрана 2022-08-31 в 15 03 20" src="https://user-images.githubusercontent.com/4948177/187674256-55ce0c71-b47a-40f0-b0ff-263a6d8eaf02.png">
<img width="1723" alt="Снимок экрана 2022-08-31 в 15 03 35" src="https://user-images.githubusercontent.com/4948177/187674272-1010b218-c958-4f4d-b83e-00a0b8302a39.png">
<img width="1732" alt="Снимок экрана 2022-08-31 в 15 03 47" src="https://user-images.githubusercontent.com/4948177/187674292-9eb785e3-f12b-49f5-95e4-b44f7cb54af2.png">
<img width="1738" alt="Снимок экрана 2022-08-31 в 15 03 59" src="https://user-images.githubusercontent.com/4948177/187674306-f61b3948-2879-4004-9a60-bbcf8b545305.png">


## Acknowledgement

- [gofish][5] provides the underlying library to interact servers

[1]: git@github.com:sbates130272/redfish_exporter.git
[2]: https://github.com/jenningsloy318/redfish_exporter/issues/7
[3]: https://prometheus.io/
[4]: https://github.com/prometheus/prometheus/wiki/Default-port-allocations
[5]: https://github.com/stmcginnis/gofish
