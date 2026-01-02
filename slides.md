---
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
hideInToc: true
title: Observability Training
author: Mischa Taylor
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# open graph
# seoMeta:
#  ogImage: https://cover.sli.dev
themeConfig:
  paginationX: r
  paginationY: t
  paginationPagesDisabled: [1]
---

# Observability Training

##### Mischa Taylor | 📧 <taylor@linux.com>

<div @click="$slidev.nav.next" class="mt-12 py-1" hover:bg="white op-10">
  Press Space for next page <carbon:arrow-right />
</div>

<div class="abs-br m-6 text-xl">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="slidev-icon-btn">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" class="slidev-icon-btn">
    <carbon:logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
hideInToc: true
routeAlias: toc
---

# Table of Contents

<Toc columns="2"/>

---
layout: section
---

# Prometheus

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

```bash
cat >prometheus.yml <<EOF
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
EOF
```

```bash
docker container run -it --rm \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/prometheus/prometheus.yml,readonly \
  --entrypoint promtool \
  docker.io/boxcutter/prometheus check config prometheus.yml
```

```bash
docker container run -it --rm \
  -p 9090:9090 \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/etc/prometheus/prometheus.yml,readonly \
  docker.io/boxcutter/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/prometheus \
    --storage.tsdb.retention.time=30d \
    --storage.tsdb.retention.size=20GB \
    --web.listen-address=:9090
```

---
hideInToc: true
---

Visit http://localhost:9090/

Metrics are served up via http://localhost:9090/metrics

Check health of targets with http://localhost:9090/targets

To use the expression browser http://localhost:9090/query

Try the following query. This expression tells you the CPU usage in cores of Prometheus itself, averaged over a 1-minute window:

`rate(process_cpu_seconds_total{job="prometheus"}[1m])`

This expression reports the resident memory usage of Prometheus itself in mebibytes (MiB):

`process_resident_memory_bytes{job="prometheus"} / 1024 / 1024`

---
hideInToc: true
---

This expression shows the number of samples per second stored by Prometheus, as averaged over a 1-minute window:

`rate(prometheus_tsdb_head_samples_appended_total{job="prometheus"}[1m])`

This is a synthetic up metric that Prometheus records for every target. In is scoped to the prometheus job:

`up{job="prometheus"}`

---
hideInToc: true
---

- **Pull-based** - Endpoints don't push out data, instead Prometheus scrapes endpoints to fetch data
- The **consumer** (Prometheus) controls the timing, not the producer
- Cannot set the scrape interval to match the rate at which you collect metrics
- Data is collected on a timer, metrics **do not** flow immediately when published
- Precision is based on best-effort, periodic sampling. It is **not** intended for high-frequency, real-time or sub-second metrics
- Metrics collection is passive, not active - Prometheus chooses when to pull the data
- Not intended to collect every event — it’s about compact summaries of what’s happening over time.

---
hideInToc: true
---

1. It Collects State, Not Raw Streams
  - Prometheus metrics are like “snapshots” of current counters or gauges.
  - Example: Instead of storing every sensor message, it stores `sensor_msgs_received_total 145723`.

 Analogy: “It’s like glancing at a dashboard every 15 seconds and writing down what the odometer says — not recording the entire drive.”

2. It’s Pull-Based and Bounded
  -You define how often Prometheus collects (scrape_interval), e.g., every 15s.
  - It only collects what’s exposed — usually a compact flat list of counters and gauges.

This puts an upper bound on CPU, memory, and I/O used by monitoring — it won’t
run wild like logs or bag files can.

---
hideInToc: true
---

3. It Stores Only Numbers and Labels
  - Prometheus stores structured numeric time series: a float64 value and a timestamp, with optional key/value tags (labels).
  - It does not store logs, strings, or individual events.

This is why Prometheus databases stay compact — even thousands of time series can be held in a few hundred MB.

4. It Uses Time and Space Efficient Storage (TSDB)
  - Prometheus uses its own highly optimized time-series database:
  - Stores data in chunks
  - Compresses samples using delta and XOR compression
  - Avoids duplication of labels
  - Prunes old data automatically (--storage.tsdb.retention.time)

 Most people can store weeks of metrics in a few GB, even across hundreds of machines.



---
hideInToc: true
---

Prometheus is designed to collect summaries of system state over time, not
high-frequency raw data.
It stores only numeric values at fixed intervals and compresses them
efficiently, so it’s great for monitoring trends, spotting failures, and
diagnosing performance issues — without overwhelming your system with logging
or bandwidth.

---
hideInToc: true
---

# Cleanup

Stop Prometheus container with <kbd>ctrl</kbd>+<kbd>c</kbd>

---
layout: section
---

# Node Exporter

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

# Exercise 02 - Your first target node_exporter

```bash
cat >prometheus.yml <<EOF
global:
  scrape_interval: 5s
  evaluation_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']

  - job_name: 'dynamic'
    file_sd_configs:
    - files:
      - /etc/prometheus/targets.yml
      - /etc/prometheus/targets.json
      refresh_interval: 30s
EOF
```

---
hideInToc: true
---

```bash
cat >targets.yml <<EOF
- targets: [ 'node-exporter:9100' ]
  labels:
    job: 'node'
EOF
```

---
hideInToc: true
---

```bash
docker volume create prometheus-data

docker network create monitoring
```

```bash
docker container run -it --rm \
  -d \
  --name prometheus \
  -p 9090:9090 \
  --network monitoring \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/etc/prometheus/prometheus.yml,readonly \
  --mount type=bind,source="$(pwd)/targets.yml",target=/etc/prometheus/targets.yml,readonly \
  --mount type=volume,source=prometheus-data,target=/prometheus,volume-driver=local \
  docker.io/boxcutter/prometheus
```

```bash
docker container run -it --rm \
  --name node-exporter \
  -p 9100:9100 \
  --network monitoring \
  docker.io/boxcutter/node-exporter \
    --collector.systemd \
    --collector.processes \
    --no-collector.infiniband \
    --no-collector.nfs \
    --web.listen-address=:9100
```

---
hideInToc: true
---

# Textfile collector

https://www.robustperception.io/using-the-textfile-collector-from-a-shell-script/

```bash
#!/bin/bash

# Adjust as needed.
TEXTFILE_COLLECTOR_DIR=/var/lib/node_exporter/textfile_collector/
# Note the start time of the script.
START="$(date +%s)"

# Your code goes here.
sleep 10

# Write out metrics to a temporary file.
END="$(date +%s)"
cat << EOF > "$TEXTFILE_COLLECTOR_DIR/myscript.prom.$$"
myscript_duration_seconds $(($END - $START))
myscript_last_run_seconds $END
EOF

# Rename the temporary file atomically.
# This avoids the node exporter seeing half a file.
mv "$TEXTFILE_COLLECTOR_DIR/myscript.prom.$$" \
  "$TEXTFILE_COLLECTOR_DIR/myscript.prom"
```

---
hideInToc: true
---

# Create a systemd service unit

```bash
# /etc/systemd/system/my_script.service
[Unit]
Description=Run my bash script

[Service]
Type=oneshot
ExecStart=/usr/local/bin/my_script.sh
```

---
hideInToc: true
---

```bash
# /etc/systemd/system/my_script.timer
[Unit]
Description=Timer for my_script.sh

[Timer]
OnCalendar=*:0/15
Persistent=true

[Install]
WantedBy=timers.target
```

That OnCalendar=*:0/15 means: every 15 minutes. You can use any systemd time expression, e.g.:
	•	daily → once a day
	•	hourly → once an hour
	•	Mon..Fri 09:00 → weekdays at 9am

---
hideInToc: true
---

Enable and start the timer
```bash
sudo systemctl daemon-reexec         # safest way to reload after edits
sudo systemctl daemon-reload
sudo systemctl enable --now my_script.timer
```

Check that it works
- See timer status: systemctl list-timers --all
- See last run: systemctl status my_script.service
- See logs: journalctl -u my_script.service

---
hideInToc: true
---

```bash
docker container stop prometheus
```

```bash
docker volume rm prometheus-data

docker network rm monitoring
```

---
layout: section
---

# Grafana

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

```bash
docker volume create prometheus-data

docker network create monitoring
```

---
hideInToc: true
---

```bash
cat >prometheus.yml <<EOF
global:
  scrape_interval: 5s
  evaluation_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']

  - job_name: 'dynamic'
    file_sd_configs:
    - files:
      - /etc/prometheus/targets.yml
      - /etc/prometheus/targets.json
      refresh_interval: 30s
EOF
```

---
hideInToc: true
---

```bash
cat >targets.yml <<EOF
- targets: [ 'node-exporter:9100' ]
  labels:
    job: 'node'
EOF
```

---
hideInToc: true
---

```bash
docker container run -it --rm \
  -d \
  --name prometheus \
  -p 9090:9090 \
  --network monitoring \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/etc/prometheus/prometheus.yml,readonly \
  --mount type=bind,source="$(pwd)/targets.yml",target=/etc/prometheus/targets.yml,readonly \
  --mount type=volume,source=prometheus-data,target=/prometheus,volume-driver=local \
  docker.io/boxcutter/prometheus
```

---
hideInToc: true
---

```bash
docker container run -it --rm \
  -d \
  --name node-exporter \
  -p 9100:9100 \
  --network monitoring \
  docker.io/boxcutter/node-exporter
```

---
hideInToc: true
---

```bash
docker volume create grafana-data
```

```bash
# https://grafana.com/docs/grafana/latest/setup-grafana/installation/docker/
docker container run -it --rm \
  --name grafana \
  -p 3000:3000 \
  --network monitoring \
  --env GF_SECURITY_ADMIN_USER=admin \
  --env GF_SECURITY_ADMIN_PASSWORD=Superseekret63 \
  --env GF_AUTH_ANONYMOUS_ENABLED=true \
  --mount type=volume,source=grafana-data,target=/var/lib/grafana,volume-driver=local \
  docker.io/grafana/grafana
```

---
hideInToc: true
---

# Set up Prometheus as a Data Source in Grafana

Navigate to Connections → Data Sources → Add data source in the left side panel.
Choose Prometheus
Prometheus server url: http://prometheus:9090
Click on the "Save & test" button

---
hideInToc: true
---
Navigate to Dashboards > New > Import
Paste in 1860 and click on the "Load" button

---
hideInToc: true
---

# Create a New Dashboard and Panel

Navigate to Dashboards → New Dashboard → Click on the Add Visualization button
Select Prometheus from the data source dropdown

---
hideInToc: true
---

```bash
docker container stop node-exporter
docker container stop prometheus
```

```bash
docker volume rm prometheus-data
docker network rm monitoring
```

---
layout: section
---

# Blackbox Exporter

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

The blackbox exporter uses the [multi-target exporter pattern](https://prometheus.io/docs/guides/multi-target-exporter/)
to monitor the availability of multiple entities from a single instance.

Multi-target exporters are used when you can't (or shouldn't) run an exporter
directly on a device.

---
hideInToc: true
---

```bash
cat >blackbox.yml <<EOF
modules:
  http_2xx:
    prober: http
    http:
      method: GET
      preferred_ip_protocol: "ip4"

  resolve_prometheus:
    prober: dns
    dns:
      query_name: prometheus.io
      query_type: A
EOF
```

---
hideInToc: true
---

```bash
docker network create monitoring
```

```bash
docker container run -it --rm \
  --name blackbox-exporter \
  -p 9115:9115 \
  --network monitoring \
  docker.io/boxcutter/blackbox_exporter
```

---
hideInToc: true
---

There is the usual `/metrics` endpoint for prometheus monitoring the health of the blackbox exporter service
itself at http://localhost:9115/metrics. But this isn't the endpoint that you generally use with the blackbox
exporter:

```bash
% curl 'localhost:9115/metrics'
```

---
hideInToc: true
---

There is another endpoint called `/probe`. A probe can generate information for another target besides the
blackbox exporter service itself. You specify a "target" and a "module" to be queried by a probe,
configured in the `blackbox.yml` file.

If you visit http://localhost:9115/ - you'll see that there are no recent probes yet. Probes only happen when
something visits the `/probe` endpoint, specifying a "target" and "module". Let's do that now:

```
% curl 'localhost:9115/probe?target=prometheus.io&module=http_2xx'
```

---
hideInToc: true
---

We can get prometheus to do the same kind probe trigger that we just did with curl. Here's
what that configuration looks like:

```
cat >prometheus.yml <<EOF
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: blackbox
    metrics_path: /probe
    params:
      module:
        - http_2xx
      target:
        - prometheus.io
    static_configs:
      - targets:
        - blackbox-exporter:9115
EOF
```

---
hideInToc: true
---

```bash
docker container run -it --rm \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/prometheus/prometheus.yml,readonly \
  --entrypoint promtool \
  docker.io/boxcutter/prometheus check config prometheus.yml
```

```bash
docker container run -it --rm \
  -d \
  --name prometheus \
  -p 9090:9090 \
  --network monitoring \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/etc/prometheus/prometheus.yml,readonly \
  docker.io/boxcutter/prometheus
```

---
hideInToc: true
---


Check http://localhost:9090/targets and verify that prometheus sent the probe successfully. It should
say the target is up.

Visit http://localhost:9090/query and perform queries for `probe_http_status_code` and `probe_http_duration_seconds`.

```
probe_http_status_code{instance="blackbox-exporter:9115", job="blackbox"}
probe_http_duration_seconds{instance="blackbox-exporter:9115", job="blackbox", phase="connect"}
```

---
hideInToc: true
---

So now we see that prometheus stores the result of a probe. There is a problem with our config. Note that the `instance`
label shows `blackbox-exporter:9115`, which is correct. But we have no idea what the name of the host that was probed,
in this case `prometheus.io`. We could look in the config file under the scrape config param to find this information,
but then there's no way to query this information later in the database.

We can fix this with relabeling. Here's what we need to know about relabeling to understand the upcoming config:

1. All labels starting with `__` are dropped after the scrape. Most internal labels start with `__`
2. There is an internal label `__address__` which is set by the targets under `static_configs` and whose value is the hostname
   for the scrape request. By default it is later used to set the value for the label `instance`, which is attached to each
   metric and tells you where the metrics came from.
3. You can set custom labels to be scrape params that are defined with `__param_<name>`.


---
hideInToc: true
---

```bash
cat >prometheus.yml <<EOF
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module:
        - http_2xx
    static_configs:
      - targets:
        - 'http://prometheus.io'     # Target to probe with HTTP.
        - 'https://prometheus.io'    # Target to probe with HTTPS.
        - 'http://promlabs.com:8080' # Unreachable target to probe with HTTP on port 8080.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115 # Blackbox exporter address.
EOF
```

---
hideInToc: true
---

So what changed in this new config?

The `target` is no longer included in the `params`:

Old:
```
params:
  module:
    - http_2xx
  target:
    - prometheus.io
```

New:
```
params:
  module:
    - http_2xx
```

---
hideInToc: true
---

This lets us include multiple targets under `static_configs` instead of just one:

Old:
```
static_configs:
  - targets:
    - blackbox-exporter:9115
```

New:
```
static_configs:
  - targets:
    - 'http://prometheus.io'     # Target to probe with HTTP.
    - 'https://prometheus.io'    # Target to probe with HTTPS.
    - 'http://promlabs.com:8080' # Unreachable target to probe with HTTP on port 8080.
```

---
hideInToc: true
---

And now we have a bunch of new relabeling rules under `relabel_configs`:

```
relabel_configs:
  - source_labels: [__address__]
    target_label: __param_target
  - source_labels: [__param_target]
    target_label: instance
  - target_label: __address__
    replacement: blackbox-exporter:9115 # Blackbox exporter address.
```

---
hideInToc: true
---

Before applying the relabeling rules, the request Prometheus would make would look like: `http://prometheus.io/probe?modules=http_2xx`.
After relabeling, the request would look like this: `http://blackbox-exporter:9115/probe?target=http://prometheus.io&module=http_2xx`.

The relabel rules are applied in sequence. First we take the values from the label `__address__`.
This comes from `targets` and write them to a new label called `__param_target`.

After this our imagined Prometheus request URI has now a target parameter: "http://prometheus.io/probe?target=http://prometheus.io&module=http_2xx".

---
hideInToc: true
---

First we take the values from the label __address__ (which contain the values from targets) and write them to a new label __param_target which will add a parameter target to the Prometheus scrape requests:

```
relabel_configs:
  - source_labels: [__address__]
    target_label: __param_target
```

After this, the first Prometheus request URI has now a target parameter: `http://prometheus.io/probe?target=http://prometheus.io&module=http_2xx`.

Next we take the values from the label `__param_target` and create a label instance with the values.

```
relabel_configs:
  - source_labels: [__param_target]
    target_label: instance
```

This relabel config won't change the request, but the metrics that come back from the request will now have a label `instance="http://prometheus.io"`.
This lets us determine which host was probed.

---
hideInToc: true
---

And finally, we write the value `blackbox-exporter:9115` to the label `__address__` (now that we have
saved a copy in the `__param_target`). This is the hostname and port where prometheus will send
the probe request: `http://blackbox-exporter:9115/probe?target=http://prometheus.io&module=http_2xx`

```
relabel_configs:
  - target_label: __address__
    replacement: blackbox-exporter:9115
```

To make things easier to copy 

```
relabel_configs:
  - source_labels: []
    target_label: instance
    replacement: localhost:9115
```

---
hideInToc: true
---

```
cat >prometheus.yml <<EOF
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module:
        - http_2xx
    static_configs:
      - targets:
        - 'http://prometheus.io'     # Target to probe with HTTP.
        - 'https://prometheus.io'    # Target to probe with HTTPS.
        - 'http://promlabs.com:8080' # Unreachable target to probe with HTTP on port 8080.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115 # Blackbox exporter address.
EOF
```

---
hideInToc: true
---

```
docker container run -it --rm \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/prometheus/prometheus.yml,readonly \
  --entrypoint promtool \
  docker.io/boxcutter/prometheus check config prometheus.yml
```

```
docker container run -it --rm \
  -d \
  --name prometheus \
  -p 9090:9090 \
  --network monitoring \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/etc/prometheus/prometheus.yml,readonly \
  --mount type=volume,source=prometheus-data,target=/prometheus,volume-driver=local \
  docker.io/boxcutter/prometheus
```

Now if you go to and query `probe_http_status_code` you should see the address of the URI that was
probed instead of `blackbox-exporter:9115`, but 

```
probe_http_status_code{instance="https://prometheus.io", job="blackbox"}
```

And for `probe_http_duration_seconds` you're still seeing valid data, so it's using `blackbox-exporter:9115`
to send the probe, and not the host listed in the instance (since it shouldn't have blackbox exporter
installed there).

---
hideInToc: true
---

```
docker stop prometheus
```

---
layout: section
---

# SNMP Exporter

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

```bash
% docker run -it --rm docker.io/boxcutter/snmp \
  snmpstatus -v3 -l authPriv \
    -u snmpreader \
    -a SHA -A superseekret \
    -x AES -X superseekret \
    10.137.56.1
[UDP: [10.137.56.1]:161->[0.0.0.0]:45090]=>[Peplink MAX BR2 Pro] Up: 1 day, 2:41:05.34
Interfaces: 9, Recv/Trans packets: 44205108/42666635 | IP: 0/0
2 interfaces are down!
```

---
hideInToc: true
---

```bash
% docker run -it --rm docker.io/boxcutter/snmp \
  snmpwalk -v3 -l authPriv \
    -u snmpreader \
    -a SHA -A superseekret \
    -x AES -X superseekret \
    10.137.56.1
iso.3.6.1.2.1.1.1.0 = STRING: "Peplink MAX BR2 Pro"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.23695
iso.3.6.1.2.1.1.3.0 = Timeticks: (28708) 0:04:47.08
iso.3.6.1.2.1.1.4.0 = STRING: "support@peplink.com"
iso.3.6.1.2.1.1.5.0 = STRING: "MAX-BR2-2013"
iso.3.6.1.2.1.1.6.0 = STRING: "Peplink"
iso.3.6.1.2.1.1.8.0 = Timeticks: (1) 0:00:00.01
...
```

---
hideInToc: true
---

```bash
git clone https://github.com/prometheus/snmp_exporter.git
cd snmp_exporter/generator
```

```bash
cp generator.yml generator.yml.orig
```

---
hideInToc: true
---

```bash
---
auths:
  public_v3:
    version: 3
    username: snmpreader
    password: superseekret
    auth_protocol: SHA
    priv_protocol: AES
    priv_password: superseekret

modules:
  # Default IF-MIB interfaces table with ifIndex.
  if_mib:
    walk: [sysUpTime, interfaces, ifXTable]
    lookups:
      - source_indexes: [ifIndex]
        lookup: "IF-MIB::ifAlias"
      - source_indexes: [ifIndex]
        # Disambiguate from PaloAlto PAN-COMMON-MIB::ifDescr.
        lookup: "IF-MIB::ifDescr"
      - source_indexes: [ifIndex]
        # Disambiguate from Netscaler NS-ROOT-MIB::ifName.
        lookup: "IF-MIB::ifName"
    overrides:
      ifAlias:
        ignore: true # Lookup metric
      ifDescr:
        ignore: true # Lookup metric
      ifName:
        ignore: true # Lookup metric
      ifType:
        type: EnumAsInfo
```

---
hideInToc: true
---

```bash
docker run -it --rm \
  --mount type=bind,source="$(pwd)",target=/code \
  --entrypoint /bin/bash \
  docker.io/boxcutter/snmp-generator
# cd /code
# generator -m /opt/snmp_exporter/mibs/mibs -m /opt/snmp_exporter/mibs/peplink/peplink generate
```

---
hideInToc: true
---

```bash
docker network create monitoring
```

```bash
docker run -it --rm \
  --name snmp-exporter \
  -p 9116:9116 \
  --network monitoring \
  --mount type=bind,source="$(pwd)/snmp.yml",target=/etc/snmp_exporter/snmp.yml \
  docker.io/boxcutter/snmp-exporter \
    --config.file=/etc/snmp_exporter/snmp.yml
```

```bash
Target: 10.137.56.1
Auth: public_v3
Module: if_mib
```

---
hideInToc: true
---

```bash
cat >prometheus.yml <<EOF
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'network'
    metrics_path: /snmp
    params:
      auth: [public_v3]
      module: [if_mib]
    static_configs:
      - targets: ['10.137.56.1']
        labels:
          job: 'peplink'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: snmp-exporter:9116 # The SNMP exporter real host
EOF
```

---
hideInToc: true
---

```bash
docker container run -it --rm \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/prometheus/prometheus.yml,readonly \
  --entrypoint promtool \
  docker.io/boxcutter/prometheus check config prometheus.yml
```

---
hideInToc: true
---

```bash
docker container run -it --rm \
  --name prometheus \
  -p 9090:9090 \
  --network monitoring \
  --mount type=bind,source="$(pwd)/prometheus.yml",target=/etc/prometheus/prometheus.yml,readonly \
  docker.io/boxcutter/prometheus
```

---
layout: section
---

# Prometheus robot fleet dashboard with flaky networks

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

If the internet link is flaky, you want two complementary signals:

	1.	“Is the robot itself up?” (local liveness)
	2.	“Can I reach it right now?” (end-to-end reachability)

Prometheus by itself mostly measures (2) unless you design for (1).

---
hideInToc: true
---

# Record “robot up” locally, then forward when the link returns

Run Prometheus on each robot (or at least a tiny agent that speaks remote_write), scrape node_exporter locally over localhost, and remote_write to your central Prometheus when it can.

Why it works: even if the robot is offline from the internet, local Prometheus still records that the computer was up; when the link comes back it ships the backlog.

What to use
- Full Prometheus on-robot + remote_write
- Or an agent (e.g., Grafana Agent / Alloy / VictoriaMetrics vmagent / OTel Collector with Prom remote_write) if you don’t want full Prometheus per robot

---
hideInToc: true
---

# Split “upness” into scrape success vs host boot health

A. Scrape success: up

Prometheus already provides:
  - up{job="node-exporter", instance="robot-123"} = 1 when scrape succeeds, 0 when it fails.

This is reachability, not necessarily “the robot is down”.

B. Host boot health: node_exporter + time drift

From node_exporter, use:
  - node_boot_time_seconds (changes only on reboot)
  - node_time_seconds (current time)
  - node_uptime_seconds (if enabled/available; otherwise derive from boot time)

In your central Prometheus, record:
  - “Last time we got any sample from this robot”
  - “Last observed boot time” (to detect reboots)

Even if you can’t get local buffering, this lets you distinguish:
  - “robot unreachable” vs “robot rebooted recently” vs “robot stable but link flaky”

---
hideInToc: true
---

# Track “last seen” explicitly (works even without buffering)

Create a recording rule that turns scrapes into a clean “last seen timestamp”:
  - For each robot/instance, compute the most recent sample time from a stable metric (boot time is good)
  - Then alert if “now - last_seen > threshold”

This is more actionable than raw up==0 because it naturally debounces brief blips.

What you’ll alert on:
  - “Not seen for 5 minutes” (warning)
  - “Not seen for 30 minutes” (critical)

And separately:
  - “Rebooted within last 10 minutes” (info/warn)

---
hideInToc: true
---

# Add an “internet health” probe from the robot side

For flaky internet, you also want: “Robot is up, but internet is bad.”

Options:
  - Run blackbox_exporter on the robot to probe:
  - DNS resolution
  - HTTPS to a known endpoint (your backend or a public endpoint you control)
  - ICMP ping to a few anchors
  - Or a lightweight cron/script that updates a textfile collector metric with:
      - packet loss %
	  - latency
	  - default route present
	  - Wi-Fi RSSI / cellular signal metrics

Then you can classify outages:
  - robot down (no local metrics even on-robot)
  - robot up but uplink down (local metrics show poor internet)
  - robot up and internet OK, but your central can’t reach (routing/firewall)

---
hideInToc: true
---

# Make identity stable so you don’t lose continuity

For a fleet, ensure your series identity doesn’t change when IPs change:
  - Set instance to something stable (robot_id + hw serial)
  - Add labels like robot_id, site, fleet, role

This makes “last seen” and “uptime” meaningful across network changes.

---
hideInToc: true
---

# Practical alerting strategy for flaky links

Don’t page on up == 0 immediately.

Use a multi-stage approach:
  - Warning: unreachable for 5–10 minutes
  - Critical: unreachable for 30–60 minutes
  - Info: reboot detected (boot time changed)
  - Correlate: if robot reports uplink loss at same time, downgrade severity

---
layout: section
---

# Alertmanager

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

---
layout: section
---

# Loki

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---
